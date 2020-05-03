Shared Informer Source Code Review
---

架构
---
```go
                        
                       => indexer                     
Reflector => FIFODelta 
                       => listener.addCh =listener.pop()=> listener.nextCh => listener.run()
                                             ↓                 ↑
                                          listener.pendingNotification（buffer）
```

核心组件
---
1. processor:    挂cache更新的钩子
2. controller:   用来更新cache
3. indexer：     cache本体
```go
func (s *sharedIndexInformer) Run(stopCh <-chan struct{}) {
	...
	wg.StartWithChannel(processorStopCh, s.cacheMutationDetector.Run)
	wg.StartWithChannel(processorStopCh, s.processor.run)
	...
	s.controller.Run(stopCh)
}
```

Controller
---
Controller的设计模式：生产者——消费者
Controller的核心逻辑：
1. 起一个reflector，用于watch对象的更新，将更新送到c.config.Queue（FIFODelta）里
2. processLoop从Queue消费对象的更新操作，同步到本地cache里(indexer)
```go
func (c *controller) Run(stopCh <-chan struct{}) {
	...

	wg.StartWithChannel(stopCh, r.Run)

	wait.Until(c.processLoop, time.Second, stopCh)
}
```

- reflector Watch对象的更新
```go
func (r *Reflector) ListAndWatch(stopCh <-chan struct{}) error {
	// List：将对象同步到本地，这一步是全量同步
	go func(){
		...
		list, paginatedResult, err = pager.List(context.Background(), metav1.ListOptions{ResourceVersion: r.relistResourceVersion()})
	    ...
		items, err := meta.ExtractList(list)
	    ...
		r.syncWith(items, resourceVersion)
	    ...
	}
	// Resync：一个单独线程用于重新同步indexer，即本地cache
	go func() {
		for {
			...
			select {
            case <-resyncCh:
            case <-stopCh:
                return
            case <-cancelCh:
                return
            }
			...
			r.store.Resync()
			...
		}
	}
	// Watch: 观察对象的变化，同步到本地，这一步是增量更新
	for {
		...
		w, err := r.listerWatcher.Watch(options)
		...
		r.watchHandler(w, &resourceVersion, resyncerrc, stopCh)
	}
}
```

- processLoop 消费更新
```go
func (c *controller) processLoop() {
	for {
		...
		obj, err := c.config.Queue.Pop(PopProcessFunc(c.config.Process))
		...
	}
}
```
Processb = HandleDeltas
```go
    Process: s.HandleDeltas,
```

- HandleDeltas 更新本地缓存，向processor发通知执行钩子函数
distribute会写notification到processor的addCh。
```go
func (s *sharedIndexInformer) HandleDeltas(obj interface{}) error {
	s.blockDeltas.Lock()
	defer s.blockDeltas.Unlock()

	// from oldest to newest
	for _, d := range obj.(Deltas) {
		switch d.Type {
		case Sync, Replaced, Added, Updated:
			s.cacheMutationDetector.AddObject(d.Object)
			if old, exists, err := s.indexer.Get(d.Object); err == nil && exists {
				if err := s.indexer.Update(d.Object); err != nil {
					return err
				}

				isSync := false
				switch {
				case d.Type == Sync:
					// Sync events are only propagated to listeners that requested resync
					isSync = true
				case d.Type == Replaced:
					if accessor, err := meta.Accessor(d.Object); err == nil {
						if oldAccessor, err := meta.Accessor(old); err == nil {
							// Replaced events that didn't change resourceVersion are treated as resync events
							// and only propagated to listeners that requested resync
							isSync = accessor.GetResourceVersion() == oldAccessor.GetResourceVersion()
						}
					}
				}
				s.processor.distribute(updateNotification{oldObj: old, newObj: d.Object}, isSync)
			} else {
				if err := s.indexer.Add(d.Object); err != nil {
					return err
				}
				s.processor.distribute(addNotification{newObj: d.Object}, false)
			}
		case Deleted:
			if err := s.indexer.Delete(d.Object); err != nil {
				return err
			}
			s.processor.distribute(deleteNotification{oldObj: d.Object}, false)
		}
	}
	return nil
}
```


Processor
---
Processor用来处理开发者挂的钩子函数。
每个listener对应一个开发者挂的eventHandler钩子函数。listener.run作为一个单独线程执行
钩子函数，listener.pop作为一个单独线程接收和缓冲执行任务。
```go
func (p *sharedProcessor) run(stopCh <-chan struct{}) {
	func() {
		...
		for _, listener := range p.listeners {
			p.wg.Start(listener.run)
			p.wg.Start(listener.pop)
		}
		...
	}()
	<-stopCh
	...
}
```

- run
执行钩子函数。
```go
func (p *processorListener) run() {
	...
    for next := range p.nextCh {
        switch notification := next.(type) {
        case updateNotification:
            p.handler.OnUpdate(notification.oldObj, notification.newObj)
        case addNotification:
            p.handler.OnAdd(notification.newObj)
        case deleteNotification:
            p.handler.OnDelete(notification.oldObj)
        default:
            utilruntime.HandleError(fmt.Errorf("unrecognized notification: %T", next))
        }
    }
    ...
}
```

- pop
```go
func (p *processorListener) pop() {
	defer utilruntime.HandleCrash()
	defer close(p.nextCh) // Tell .run() to stop

	var nextCh chan<- interface{}
	var notification interface{}
	for {
		select {
		case nextCh <- notification:
			// Notification dispatched
			var ok bool
			notification, ok = p.pendingNotifications.ReadOne()
			if !ok { // Nothing to pop
				nextCh = nil // Disable this select case
			}
		case notificationToAdd, ok := <-p.addCh:
			if !ok {
				return
			}
			if notification == nil { // No notification to pop (and pendingNotifications is empty)
				// Optimize the case - skip adding to pendingNotifications
				notification = notificationToAdd
				nextCh = p.nextCh
			} else { // There is already a notification waiting to be dispatched
				p.pendingNotifications.WriteOne(notificationToAdd)
			}
		}
	}
}
```