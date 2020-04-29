Deployment Controller Source Code Review
---

From controller entry
---
Deployment控制器会起几个worker协程按每秒一次的频率出来deployment item。
```go
// Run begins watching and syncing.
func (dc *DeploymentController) Run(workers int, stopCh <-chan struct{}) {
	defer utilruntime.HandleCrash()
	defer dc.queue.ShutDown()

	klog.Infof("Starting deployment controller")
	defer klog.Infof("Shutting down deployment controller")

	if !cache.WaitForNamedCacheSync("deployment", stopCh, dc.dListerSynced, dc.rsListerSynced, dc.podListerSynced) {
		return
	}
	
	for i := 0; i < workers; i++ {
		go wait.Until(dc.worker, time.Second, stopCh)
	}

	<-stopCh
}
```

Worker 
---
Worker实际上就是循环的处理下一个deployment item。
```go
func (dc *DeploymentController) worker() {
	for dc.processNextWorkItem() {
	}
}
```

Process deployment item
---
Deployments对象入队controller的队列。这是一个限流队列。controller取下一个队列元素进行处理。
```go
func (dc *DeploymentController) processNextWorkItem() bool {
	key, quit := dc.queue.Get()
	if quit {
		return false
	}
	defer dc.queue.Done(key)

	err := dc.syncHandler(key.(string))
	dc.handleErr(err, key)

	return true
}
```

Sync deployments
---
syncDeployment会对deployment做三类操作：
- rollback
- recreate
- rollout
```go
// syncDeployment will sync the deployment with the given key.
// This function is not meant to be invoked concurrently with the same key.
func (dc *DeploymentController) syncDeployment(key string) error {
	startTime := time.Now()
	klog.V(4).Infof("Started syncing deployment %q (%v)", key, startTime)
	defer func() {
		klog.V(4).Infof("Finished syncing deployment %q (%v)", key, time.Since(startTime))
	}()

    // 这一步找到对应deployment
	namespace, name, err := cache.SplitMetaNamespaceKey(key)
	if err != nil {
		return err
	}
	deployment, err := dc.dLister.Deployments(namespace).Get(name)
	if errors.IsNotFound(err) {
		klog.V(2).Infof("Deployment %v has been deleted", key)
		return nil
	}
	if err != nil {
		return err
	}

	// Deep-copy otherwise we are mutating our cache.
	d := deployment.DeepCopy()

	everything := metav1.LabelSelector{}
	if reflect.DeepEqual(d.Spec.Selector, &everything) {
		dc.eventRecorder.Eventf(d, v1.EventTypeWarning, "SelectingAll", "This deployment is selecting all pods. A non-empty selector is required.")
		// Update generation number
		if d.Status.ObservedGeneration < d.Generation {
			d.Status.ObservedGeneration = d.Generation
			dc.client.AppsV1().Deployments(d.Namespace).UpdateStatus(context.TODO(), d, metav1.UpdateOptions{})
		}
		return nil
	}

	// List ReplicaSets owned by this Deployment, while reconciling ControllerRef
	// through adoption/orphaning.
	rsList, err := dc.getReplicaSetsForDeployment(d)
	if err != nil {
		return err
	}
	// List all Pods owned by this Deployment, grouped by their ReplicaSet.
	// Current uses of the podMap are:
	//
	// * check if a Pod is labeled correctly with the pod-template-hash label.
	// * check that no old Pods are running in the middle of Recreate Deployments.
	podMap, err := dc.getPodMapForDeployment(d, rsList)
	if err != nil {
		return err
	}

	if d.DeletionTimestamp != nil {
		return dc.syncStatusOnly(d, rsList)
	}

	// Update deployment conditions with an Unknown condition when pausing/resuming
	// a deployment. In this way, we can be sure that we won't timeout when a user
	// resumes a Deployment with a set progressDeadlineSeconds.
	if err = dc.checkPausedConditions(d); err != nil {
		return err
	}

	if d.Spec.Paused {
		return dc.sync(d, rsList)
	}

	// rollback is not re-entrant in case the underlying replica sets are updated with a new
	// revision so we should ensure that we won't proceed to update replica sets until we
	// make sure that the deployment has cleaned up its rollback spec in subsequent enqueues.
	if getRollbackTo(d) != nil {
		return dc.rollback(d, rsList)
	}

	scalingEvent, err := dc.isScalingEvent(d, rsList)
	if err != nil {
		return err
	}
	if scalingEvent {
		return dc.sync(d, rsList)
	}

	switch d.Spec.Strategy.Type {
	case apps.RecreateDeploymentStrategyType:
		return dc.rolloutRecreate(d, rsList, podMap)
	case apps.RollingUpdateDeploymentStrategyType:
		return dc.rolloutRolling(d, rsList)
	}
	return fmt.Errorf("unexpected deployment strategy type: %s", d.Spec.Strategy.Type)
}
```

Rollback
---
```go
// rollback the deployment to the specified revision. In any case cleanup the rollback spec.
func (dc *DeploymentController) rollback(d *apps.Deployment, rsList []*apps.ReplicaSet) error {
	// 首先区分新旧replica set
	newRS, allOldRSs, err := dc.getAllReplicaSetsAndSyncRevision(d, rsList, true)
	if err != nil {
		return err
	}

	allRSs := append(allOldRSs, newRS)
	rollbackTo := getRollbackTo(d)
	// If rollback revision is 0, rollback to the last revision
	// 回滚版本为0时，回滚到最新版本
	if rollbackTo.Revision == 0 {
		if rollbackTo.Revision = deploymentutil.LastRevision(allRSs); rollbackTo.Revision == 0 {
			// If we still can't find the last revision, gives up rollback
			dc.emitRollbackWarningEvent(d, deploymentutil.RollbackRevisionNotFound, "Unable to find last revision.")
			// Gives up rollback
			return dc.updateDeploymentAndClearRollbackTo(d)
		}
	}
	for _, rs := range allRSs {
		v, err := deploymentutil.Revision(rs)
		if err != nil {
			klog.V(4).Infof("Unable to extract revision from deployment's replica set %q: %v", rs.Name, err)
			continue
		}
		// 找到要回滚的版本，然后执行回滚
		if v == rollbackTo.Revision {
			klog.V(4).Infof("Found replica set %q with desired revision %d", rs.Name, v)
			// rollback by copying podTemplate.Spec from the replica set
			// revision number will be incremented during the next getAllReplicaSetsAndSyncRevision call
			// no-op if the spec matches current deployment's podTemplate.Spec
			performedRollback, err := dc.rollbackToTemplate(d, rs)
			if performedRollback && err == nil {
				dc.emitRollbackNormalEvent(d, fmt.Sprintf("Rolled back deployment %q to revision %d", d.Name, rollbackTo.Revision))
			}
			return err
		}
	}
	dc.emitRollbackWarningEvent(d, deploymentutil.RollbackRevisionNotFound, "Unable to find the revision to rollback to.")
	// Gives up rollback
	return dc.updateDeploymentAndClearRollbackTo(d)
}
```
看下rollback细节，如果要回滚的版本template和当前不一样，则设置template和annotations。
如果template一致，会避免rollback。
```go
func (dc *DeploymentController) rollbackToTemplate(d *apps.Deployment, rs *apps.ReplicaSet) (bool, error) {
	performedRollback := false
	if !deploymentutil.EqualIgnoreHash(&d.Spec.Template, &rs.Spec.Template) {
		klog.V(4).Infof("Rolling back deployment %q to template spec %+v", d.Name, rs.Spec.Template.Spec)
		deploymentutil.SetFromReplicaSetTemplate(d, rs.Spec.Template)
		deploymentutil.SetDeploymentAnnotationsTo(d, rs)
		performedRollback = true
	} else {
		klog.V(4).Infof("Rolling back to a revision that contains the same template as current deployment %q, skipping rollback...", d.Name)
		eventMsg := fmt.Sprintf("The rollback revision contains the same template as current deployment %q", d.Name)
		dc.emitRollbackWarningEvent(d, deploymentutil.RollbackTemplateUnchanged, eventMsg)
	}

	return performedRollback, dc.updateDeploymentAndClearRollbackTo(d)
}
```
对deployment更新使用HTTP client的Update方法，实际就是发一个HTTP请求。
```go
func (dc *DeploymentController) updateDeploymentAndClearRollbackTo(d *apps.Deployment) error {
	klog.V(4).Infof("Cleans up rollbackTo of deployment %q", d.Name)
	setRollbackTo(d, nil)
	_, err := dc.client.AppsV1().Deployments(d.Namespace).Update(context.TODO(), d, metav1.UpdateOptions{})
	return err
}
```

Recreate
---
Recreate是先清空旧RS实例，再新建新RS
```go
func (dc *DeploymentController) rolloutRecreate(d *apps.Deployment, rsList []*apps.ReplicaSet, podMap map[types.UID][]*v1.Pod) error {
	// Don't create a new RS if not already existed, so that we avoid scaling up before scaling down.
	// 找到新旧RS
	newRS, oldRSs, err := dc.getAllReplicaSetsAndSyncRevision(d, rsList, false)
	if err != nil {
		return err
	}
	allRSs := append(oldRSs, newRS)
	// 找到依旧活跃的旧RS
	activeOldRSs := controller.FilterActiveReplicaSets(oldRSs)

	// 把所有旧的RS scale down，实际就是实例数量降为0
	scaledDown, err := dc.scaleDownOldReplicaSetsForRecreate(activeOldRSs, d)
	if err != nil {
		return err
	}
	if scaledDown {
		// Update DeploymentStatus.
		return dc.syncRolloutStatus(allRSs, newRS, d)
	}

	// Do not process a deployment when it has old pods running.
	if oldPodsRunning(newRS, oldRSs, podMap) {
		return dc.syncRolloutStatus(allRSs, newRS, d)
	}

	// If we need to create a new RS, create it now.
	if newRS == nil {
		newRS, oldRSs, err = dc.getAllReplicaSetsAndSyncRevision(d, rsList, true)
		if err != nil {
			return err
		}
		allRSs = append(oldRSs, newRS)
	}

	// scale up new replica set.
	// 新建新RS
	if _, err := dc.scaleUpNewReplicaSetForRecreate(newRS, d); err != nil {
		return err
	}

	if util.DeploymentComplete(d, &d.Status) {
		if err := dc.cleanupDeployment(oldRSs, d); err != nil {
			return err
		}
	}

	// Sync deployment status.
	return dc.syncRolloutStatus(allRSs, newRS, d)
}
```

Rollout
---
Rollout是先把新RS起起来，然后删旧RS
```go
func (dc *DeploymentController) rolloutRolling(d *apps.Deployment, rsList []*apps.ReplicaSet) error {
	newRS, oldRSs, err := dc.getAllReplicaSetsAndSyncRevision(d, rsList, true)
	if err != nil {
		return err
	}
	allRSs := append(oldRSs, newRS)

	// Scale up, if we can.
	scaledUp, err := dc.reconcileNewReplicaSet(allRSs, newRS, d)
	if err != nil {
		return err
	}
	if scaledUp {
		// Update DeploymentStatus
		return dc.syncRolloutStatus(allRSs, newRS, d)
	}

	// Scale down, if we can.
	scaledDown, err := dc.reconcileOldReplicaSets(allRSs, controller.FilterActiveReplicaSets(oldRSs), newRS, d)
	if err != nil {
		return err
	}
	if scaledDown {
		// Update DeploymentStatus
		return dc.syncRolloutStatus(allRSs, newRS, d)
	}

	if deploymentutil.DeploymentComplete(d, &d.Status) {
		if err := dc.cleanupDeployment(oldRSs, d); err != nil {
			return err
		}
	}

	// Sync deployment status
	return dc.syncRolloutStatus(allRSs, newRS, d)
}
```