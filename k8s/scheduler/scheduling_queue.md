Scheduling Queue in k8s
---

What is a scheduling queue?
---
Kubernetes Scheduler use a scheduling queue to hold pods to schedule. 
```go
// PriorityQueue implements a scheduling queue.
// The head of PriorityQueue is the highest priority pending pod. This structure
// has three sub queues. One sub-queue holds pods that are being considered for
// scheduling. This is called activeQ and is a Heap. Another queue holds
// pods that are already tried and are determined to be unschedulable. The latter
// is called unschedulableQ. The third queue holds pods that are moved from
// unschedulable queues and will be moved to active queue when backoff are completed.
type PriorityQueue struct {
	stop  chan struct{}
	clock util.Clock

	// pod initial backoff duration.
	podInitialBackoffDuration time.Duration
	// pod maximum backoff duration.
	podMaxBackoffDuration time.Duration

	lock sync.RWMutex
	cond sync.Cond

	// activeQ is heap structure that scheduler actively looks at to find pods to
	// schedule. Head of heap is the highest priority pod.
	activeQ *heap.Heap
	// podBackoffQ is a heap ordered by backoff expiry. Pods which have completed backoff
	// are popped from this heap before the scheduler looks at activeQ
	podBackoffQ *heap.Heap
	// unschedulableQ holds pods that have been tried and determined unschedulable.
	unschedulableQ *UnschedulablePodsMap
	// nominatedPods is a structures that stores pods which are nominated to run
	// on nodes.
	nominatedPods *nominatedPodMap
	// schedulingCycle represents sequence number of scheduling cycle and is incremented
	// when a pod is popped.
	schedulingCycle int64
	// moveRequestCycle caches the sequence number of scheduling cycle when we
	// received a move request. Unscheduable pods in and before this scheduling
	// cycle will be put back to activeQueue if we were trying to schedule them
	// when we received move request.
	moveRequestCycle int64

	// closed indicates that the queue is closed.
	// It is mainly used to let Pop() exit its control loop while waiting for an item.
	closed bool
}
```

Why back off?
---
It seems that we only need two queues, active and unschedulable, at first glance. But 
imagine that we have a pod at the head of the queue and it is tried and determined 
unschedulable. We then put it back in the queue. Assume no pods with higher priority 
join so we need to schedule the same pod the second times, the thirds times and so on...
As long as there's no pod with higher priority joins and the pod is unschedulable for 
some reason, we are trapped in a dead loop. This could starves all other scheduable pods 
in the queue.

To avoid pods starvation, pods scheduled failed are backed off for a certain period 
calculated according to:
```go
func (p *PriorityQueue) calculateBackoffDuration(podInfo *framework.PodInfo) time.Duration {
	duration := p.podInitialBackoffDuration
	for i := 1; i < podInfo.Attempts; i++ {
		duration = duration * 2
		if duration > p.podMaxBackoffDuration {
			return p.podMaxBackoffDuration
		}
	}
	return duration
}
```

How scheduling queue run?
---
**- scheduling queue manager**

When we start the scheduler, it starts the scheduling queue and starts scheduling until the context is closed or expired.
```go
func (sched *Scheduler) Run(ctx context.Context) {
	if !cache.WaitForCacheSync(ctx.Done(), sched.scheduledPodsHasSynced) {
		return
	}
	sched.SchedulingQueue.Run()
	wait.UntilWithContext(ctx, sched.scheduleOne, 0)
	sched.SchedulingQueue.Close()
}
```
The method SchedulingQueue.Run() starts a scheduling queue manager which:<br/>
(1)periodically(1 second) flush all completed backoff pods to active queue<br/>
(2)periodically(30 seconds) flush unschedulable pods to active queue or backoff queue. Pods are flushed to active queue only if they have completed backoff.

```go
// Run starts the goroutine to pump from podBackoffQ to activeQ
func (p *PriorityQueue) Run() {
	go wait.Until(p.flushBackoffQCompleted, 1.0*time.Second, p.stop)
	go wait.Until(p.flushUnschedulableQLeftover, 30*time.Second, p.stop)
}
```

**- Pop**

Because scheduler polls at its scheduling queue waiting for pods to schedule, the pop 
operation is blocking.
```go
func (sched *Scheduler) scheduleOne(ctx context.Context) {
	podInfo := sched.NextPod()
	...
}
```
NextPod() is a function wrapping around Pop().
```go
func (p *PriorityQueue) Pop() (*framework.PodInfo, error) {
	p.lock.Lock()
	defer p.lock.Unlock()
	for p.activeQ.Len() == 0 {
		// When the queue is empty, invocation of Pop() is blocked until new item is enqueued.
		// When Close() is called, the p.closed is set and the condition is broadcast,
		// which causes this loop to continue and return from the Pop().
		if p.closed {
			return nil, fmt.Errorf(queueClosed)
		}
		p.cond.Wait()
	}
	obj, err := p.activeQ.Pop()
	if err != nil {
		return nil, err
	}
	pInfo := obj.(*framework.PodInfo)
	pInfo.Attempts++
	p.schedulingCycle++
	return pInfo, err
}
```
If the activeQ is empty, the current goroutine will be suspended by p.cond.Wait().
TODO: read src/sync/cond.go
```go

func (c *Cond) Wait() {
	c.checker.check()
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}
```
