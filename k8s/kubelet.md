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
