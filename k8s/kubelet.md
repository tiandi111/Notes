Kubelet Source Code Review
---

启动前的配置代码暂时省略...

Start Kubelet
---
这里做两部分工作，运行kubelet以及启动kubelet HTTP服务器。
```go
func startKubelet(k kubelet.Bootstrap, podCfg *config.PodConfig, kubeCfg *kubeletconfiginternal.KubeletConfiguration, kubeDeps *kubelet.Dependencies, enableCAdvisorJSONEndpoints, enableServer bool) {
	// start the kubelet
	go wait.Until(func() {
		k.Run(podCfg.Updates())
	}, 0, wait.NeverStop)

	// start the kubelet server
	if enableServer {
		go k.ListenAndServe(net.ParseIP(kubeCfg.Address), uint(kubeCfg.Port), kubeDeps.TLSOptions, kubeDeps.Auth, enableCAdvisorJSONEndpoints, kubeCfg.EnableDebuggingHandlers, kubeCfg.EnableContentionProfiling)

	}
	if kubeCfg.ReadOnlyPort > 0 {
		go k.ListenAndServeReadOnly(net.ParseIP(kubeCfg.Address), uint(kubeCfg.ReadOnlyPort), enableCAdvisorJSONEndpoints)
	}
	if utilfeature.DefaultFeatureGate.Enabled(features.KubeletPodResources) {
		go k.ListenAndServePodResources()
	}
}
```

Run Kubelet
---
Kubelet启动流程：
1. 启动云资源同步管理协程cloudResourceSyncManager，主要工作是从云提供商同步节点地址。
2. 初始化不依赖于容器运行时的一些模块，包括文件路径，镜像管理，服务器证书管理，oom监控，资源分析等。
3. 启动volume管理器协程
4. 启动节点状态同步协程
5. 启动节点租约（lease）控制器，用于心跳检测？
6. 检测Network，Runtime是否ready，启动依赖于容器运行时的模块：启动cAdvisor，获取node结构体，启动
containerManager，evictionManager，containerLogManager，pluginManager。
7. 配置iptablesguize
8. 启动podKiller协程
9. 启动status manager
10. 启动probing manager
11. 启动pleg generator（pod lifecycle event generator）
12. 核心流程syncLoop
```go
func (kl *Kubelet) Run(updates <-chan kubetypes.PodUpdate) {
	if kl.logServer == nil {
		kl.logServer = http.StripPrefix("/logs/", http.FileServer(http.Dir("/var/log/")))
	}
	if kl.kubeClient == nil {
		klog.Warning("No api server defined - no node status update will be sent.")
	}

	// Start the cloud provider sync manager
	if kl.cloudResourceSyncManager != nil {
		go kl.cloudResourceSyncManager.Run(wait.NeverStop)
	}

	if err := kl.initializeModules(); err != nil {
		kl.recorder.Eventf(kl.nodeRef, v1.EventTypeWarning, events.KubeletSetupFailed, err.Error())
		klog.Fatal(err)
	}

	// Start volume manager
	go kl.volumeManager.Run(kl.sourcesReady, wait.NeverStop)

	if kl.kubeClient != nil {
		// Start syncing node status immediately, this may set up things the runtime needs to run.
		go wait.Until(kl.syncNodeStatus, kl.nodeStatusUpdateFrequency, wait.NeverStop)
		go kl.fastStatusUpdateOnce()

		// start syncing lease
		go kl.nodeLeaseController.Run(wait.NeverStop)
	}
	go wait.Until(kl.updateRuntimeUp, 5*time.Second, wait.NeverStop)

	// Set up iptables util rules
	if kl.makeIPTablesUtilChains {
		kl.initNetworkUtil()
	}

	// Start a goroutine responsible for killing pods (that are not properly
	// handled by pod workers).
	go wait.Until(kl.podKiller, 1*time.Second, wait.NeverStop)

	// Start component sync loops.
	kl.statusManager.Start()
	kl.probeManager.Start()

	// Start syncing RuntimeClasses if enabled.
	if kl.runtimeClassManager != nil {
		kl.runtimeClassManager.Start(wait.NeverStop)
	}

	// Start the pod lifecycle event generator.
	kl.pleg.Start()
	kl.syncLoop(updates, kl)
}
```

- cloudResourceSyncManger<br/>
云资源管理协程的主要工作是同步节点地址
TODO: 这里的地址是虚拟机的地址？如果云提供商提供的是容器？容器上可不可以起新的容器？
```go
// Run starts the cloud resource sync manager's sync loop.
func (m *cloudResourceSyncManager) Run(stopCh <-chan struct{}) {
	wait.Until(m.syncNodeAddresses, m.syncPeriod, stopCh)
}
```

- initializeModules
暂略

- volumeManager
see kubelet_volume_manager.md

- syncNodeStatus
1. 向api server注册当前节点
2. 更新节点状态: CIDR(节点IP范围), NodeAddress, MachineInfo, GetDevicePluginResourceCapacity,
VersionInfo, DaemonEndpoints, Images, GoRuntime, VolumeLimits, MemoryPressureCondition,
DiskPressureCondition, PIDPressureCondition, ReadyCondition, VolumesInUse,
recordNodeSchedulableEvent等。 see k8s.io\kubernetes\pkg\kubelet\kubelet_node_status.go
```go
func (kl *Kubelet) syncNodeStatus() {
	kl.syncNodeStatusMux.Lock()
	defer kl.syncNodeStatusMux.Unlock()

	if kl.kubeClient == nil || kl.heartbeatClient == nil {
		return
	}
	if kl.registerNode {
		// This will exit immediately if it doesn't need to do anything.
		kl.registerWithAPIServer()
	}
	if err := kl.updateNodeStatus(); err != nil {
		klog.Errorf("Unable to update node status: %v", err)
	}
}
```

- fastStatusUpdateOnce<br/>
监控CIDR的变化，只要其改变，就立即更新一次node状态

- nodeLeaseController
see node_lease_controller.md

- updateRuntimeUp

- podKiller
```go
func (kl *Kubelet) podKiller() {
	killing := sets.NewString()
	// guard for the killing set
	lock := sync.Mutex{}
	// 要删的pod从一个channel送来
	for podPair := range kl.podKillingCh {
		runningPod := podPair.RunningPod
		apiPod := podPair.APIPod

		lock.Lock()
		exists := killing.Has(string(runningPod.ID))
		if !exists {
			killing.Insert(string(runningPod.ID))
		}
		lock.Unlock()

        // 每一个要删的pod都起一个协程处理
		if !exists {
			go func(apiPod *v1.Pod, runningPod *kubecontainer.Pod) {
				klog.V(2).Infof("Killing unwanted pod %q", runningPod.Name)
				err := kl.killPod(apiPod, runningPod, nil, nil)
				if err != nil {
					klog.Errorf("Failed killing the pod %q: %v", runningPod.Name, err)
				}
				lock.Lock()
				killing.Delete(string(runningPod.ID))
				lock.Unlock()
			}(apiPod, runningPod)
		}
	}
}
```
killPod核心逻辑TODO：QOS等级、cgroups底层机制
```go
func (kl *Kubelet) killPod(pod *v1.Pod, runningPod *kubecontainer.Pod, status *kubecontainer.PodStatus, gracePeriodOverride *int64) error {
	var p kubecontainer.Pod
	if runningPod != nil {
		p = *runningPod
	} else if status != nil {
		p = kubecontainer.ConvertPodStatusToRunningPod(kl.getRuntime().Type(), status)
	} else {
		return fmt.Errorf("one of the two arguments must be non-nil: runningPod, status")
	}
    // 主要是两件事：杀掉pod，更新Qos cgroups
	// Call the container runtime KillPod method which stops all running containers of the pod
	if err := kl.containerRuntime.KillPod(pod, p, gracePeriodOverride); err != nil {
		return err
	}
	if err := kl.containerManager.UpdateQOSCgroups(); err != nil {
		klog.V(2).Infof("Failed to update QoS cgroups while killing pod: %v", err)
	}
	return nil
}
```
- status manager

- probing manager

- pleg generator（pod lifecycle event generator）

- syncLoop
```go
func (kl *Kubelet) syncLoop(updates <-chan kubetypes.PodUpdate, handler SyncHandler) {
	klog.Info("Starting kubelet main sync loop.")
	// The syncTicker wakes up kubelet to checks if there are any pod workers
	// that need to be sync'd. A one-second period is sufficient because the
	// sync interval is defaulted to 10s.
	syncTicker := time.NewTicker(time.Second)
	defer syncTicker.Stop()
	housekeepingTicker := time.NewTicker(housekeepingPeriod)
	defer housekeepingTicker.Stop()
	plegCh := kl.pleg.Watch()
	const (
		base   = 100 * time.Millisecond
		max    = 5 * time.Second
		factor = 2
	)
	duration := base
	// Responsible for checking limits in resolv.conf
	// The limits do not have anything to do with individual pods
	// Since this is called in syncLoop, we don't need to call it anywhere else
	if kl.dnsConfigurer != nil && kl.dnsConfigurer.ResolverConfig != "" {
		kl.dnsConfigurer.CheckLimitsForResolvConf()
	}

	for {
		// runtime报错时会跳过pod同步，如果持续报错，则按指数级延长睡眠时间
		if err := kl.runtimeState.runtimeErrors(); err != nil {
			klog.Errorf("skipping pod synchronization - %v", err)
			// exponential backoff
			time.Sleep(duration)
			duration = time.Duration(math.Min(float64(max), factor*float64(duration)))
			continue
		}
		// reset backoff if we have a success
		duration = base
        
		// Store提供原子性的数据更改，syncLoopMonitor是一个atomic.Value类型
		kl.syncLoopMonitor.Store(kl.clock.Now())
		// 核心处理逻辑
		if !kl.syncLoopIteration(updates, handler, syncTicker.C, housekeepingTicker.C, plegCh) {
			break
		}
		kl.syncLoopMonitor.Store(kl.clock.Now())
	}
}
```
syncLoop的核心逻辑是一个select语句，读取多个不同的channel，执行相应的handler。包括：
- configCh:         读取配置事件，包括ADD，UPDATE，REMOVE，RECONCILE，DELETE，RESTORE，SET等
- syncCh:           读取pod同步事件，并同步pod
- housekeepingCh:   执行pod清理的事件
- plegCh:           a channel to read PLEG updates from
- liveness manager: 如果有failed的pod，或者未通过liveness check的container，执行同步
```go
func (kl *Kubelet) syncLoopIteration(configCh <-chan kubetypes.PodUpdate, handler SyncHandler,
	syncCh <-chan time.Time, housekeepingCh <-chan time.Time, plegCh <-chan *pleg.PodLifecycleEvent) bool {
	select {
	case u, open := <-configCh:
		// Update from a config source; dispatch it to the right handler
		// callback.
		if !open {
			klog.Errorf("Update channel is closed. Exiting the sync loop.")
			return false
		}

		switch u.Op {
		case kubetypes.ADD:
			klog.V(2).Infof("SyncLoop (ADD, %q): %q", u.Source, format.Pods(u.Pods))
			// After restarting, kubelet will get all existing pods through
			// ADD as if they are new pods. These pods will then go through the
			// admission process and *may* be rejected. This can be resolved
			// once we have checkpointing.
			handler.HandlePodAdditions(u.Pods)
		case kubetypes.UPDATE:
			klog.V(2).Infof("SyncLoop (UPDATE, %q): %q", u.Source, format.PodsWithDeletionTimestamps(u.Pods))
			handler.HandlePodUpdates(u.Pods)
		case kubetypes.REMOVE:
			klog.V(2).Infof("SyncLoop (REMOVE, %q): %q", u.Source, format.Pods(u.Pods))
			handler.HandlePodRemoves(u.Pods)
		case kubetypes.RECONCILE:
			klog.V(4).Infof("SyncLoop (RECONCILE, %q): %q", u.Source, format.Pods(u.Pods))
			handler.HandlePodReconcile(u.Pods)
		case kubetypes.DELETE:
			klog.V(2).Infof("SyncLoop (DELETE, %q): %q", u.Source, format.Pods(u.Pods))
			// DELETE is treated as a UPDATE because of graceful deletion.
			handler.HandlePodUpdates(u.Pods)
		case kubetypes.RESTORE:
			klog.V(2).Infof("SyncLoop (RESTORE, %q): %q", u.Source, format.Pods(u.Pods))
			// These are pods restored from the checkpoint. Treat them as new
			// pods.
			handler.HandlePodAdditions(u.Pods)
		case kubetypes.SET:
			// TODO: Do we want to support this?
			klog.Errorf("Kubelet does not support snapshot update")
		}

		if u.Op != kubetypes.RESTORE {
			// If the update type is RESTORE, it means that the update is from
			// the pod checkpoints and may be incomplete. Do not mark the
			// source as ready.

			// Mark the source ready after receiving at least one update from the
			// source. Once all the sources are marked ready, various cleanup
			// routines will start reclaiming resources. It is important that this
			// takes place only after kubelet calls the update handler to process
			// the update to ensure the internal pod cache is up-to-date.
			kl.sourcesReady.AddSource(u.Source)
		}
	case e := <-plegCh:
		if isSyncPodWorthy(e) {
			// PLEG event for a pod; sync it.
			if pod, ok := kl.podManager.GetPodByUID(e.ID); ok {
				klog.V(2).Infof("SyncLoop (PLEG): %q, event: %#v", format.Pod(pod), e)
				handler.HandlePodSyncs([]*v1.Pod{pod})
			} else {
				// If the pod no longer exists, ignore the event.
				klog.V(4).Infof("SyncLoop (PLEG): ignore irrelevant event: %#v", e)
			}
		}

		if e.Type == pleg.ContainerDied {
			if containerID, ok := e.Data.(string); ok {
				kl.cleanUpContainersInPod(e.ID, containerID)
			}
		}
	case <-syncCh:
		// Sync pods waiting for sync
		podsToSync := kl.getPodsToSync()
		if len(podsToSync) == 0 {
			break
		}
		klog.V(4).Infof("SyncLoop (SYNC): %d pods; %s", len(podsToSync), format.Pods(podsToSync))
		handler.HandlePodSyncs(podsToSync)
	case update := <-kl.livenessManager.Updates():
		if update.Result == proberesults.Failure {
			// The liveness manager detected a failure; sync the pod.

			// We should not use the pod from livenessManager, because it is never updated after
			// initialization.
			pod, ok := kl.podManager.GetPodByUID(update.PodUID)
			if !ok {
				// If the pod no longer exists, ignore the update.
				klog.V(4).Infof("SyncLoop (container unhealthy): ignore irrelevant update: %#v", update)
				break
			}
			klog.V(1).Infof("SyncLoop (container unhealthy): %q", format.Pod(pod))
			handler.HandlePodSyncs([]*v1.Pod{pod})
		}
	case <-housekeepingCh:
		if !kl.sourcesReady.AllReady() {
			// If the sources aren't ready or volume manager has not yet synced the states,
			// skip housekeeping, as we may accidentally delete pods from unready sources.
			klog.V(4).Infof("SyncLoop (housekeeping, skipped): sources aren't ready yet.")
		} else {
			klog.V(4).Infof("SyncLoop (housekeeping)")
			if err := handler.HandlePodCleanups(); err != nil {
				klog.Errorf("Failed cleaning pods: %v", err)
			}
		}
	}
	return true
}
```