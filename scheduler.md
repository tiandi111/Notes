#Scheduler

## Scheduler结构体
   // Scheduler watches for new unscheduled pods. It attempts to find
   // nodes that they fit on and writes bindings back to the api server.
   ```
   type Scheduler struct {
   	// It is expected that changes made via SchedulerCache will be observed
   	// by NodeLister and Algorithm.
   	SchedulerCache internalcache.Cache
   
   	Algorithm core.ScheduleAlgorithm
   	// PodConditionUpdater is used only in case of scheduling errors. If we succeed
   	// with scheduling, PodScheduled condition will be updated in apiserver in /bind
   	// handler so that binding and setting PodCondition it is atomic.
   	podConditionUpdater podConditionUpdater
   	// PodPreemptor is used to evict pods and update 'NominatedNode' field of
   	// the preemptor pod.
   	podPreemptor podPreemptor
   
   	// NextPod should be a function that blocks until the next pod
   	// is available. We don't use a channel for this, because scheduling
   	// a pod may take some amount of time and we don't want pods to get
   	// stale while they sit in a channel.
   	NextPod func() *framework.PodInfo
   
   	// Error is called if there is an error. It is passed the pod in
   	// question, and the error
   	Error func(*framework.PodInfo, error)
   
   	// Close this to shut down the scheduler.
   	StopEverything <-chan struct{}
   
   	// VolumeBinder handles PVC/PV binding for the pod.
   	VolumeBinder scheduling.SchedulerVolumeBinder
   
   	// Disable pod preemption or not.
   	DisablePreemption bool
   
   	// SchedulingQueue holds pods to be scheduled
   	SchedulingQueue internalqueue.SchedulingQueue
   
   	// Profiles are the scheduling profiles.
   	Profiles profile.Map
   
   	scheduledPodsHasSynced func() bool
   }
   ```
## New

##genericScheduler(默认)

### Schedule() 
主流程及解释
```go
func (g *genericScheduler) Schedule(ctx context.Context, prof *profile.Profile, state *framework.CycleState, pod *v1.Pod) (result ScheduleResult, err error) {
	// PVC检验
	err := podPassesBasicChecks(pod, g.pvcLister) 
	// 更新Cache
	err := g.snapshot()
	...
	preFilterStatus := prof.RunPreFilterPlugins(ctx, state, pod)
	// 预选阶段
	// 1. node的选取采用round-robin方式
	// 2. 并发预选
	filteredNodes, filteredNodesStatuses, err := g.findNodesThatFitPod(ctx, prof, state, pod) 
	...
	prescoreStatus := prof.RunPreScorePlugins(ctx, state, pod, filteredNodes)
	// 优选阶段
	// 1. 并发执行
	priorityList, err := g.prioritizeNodes(ctx, prof, state, pod, filteredNodes)
	// 终选，蓄水池算法（等概率抽取）
	host, err := g.selectHost(priorityList)
	...
}
```