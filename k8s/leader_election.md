Leader Election in K8s
---

## Summary


## 核心逻辑
```go
// Run starts the leader election loop
func (le *LeaderElector) Run(ctx context.Context) {
	defer func() {
		runtime.HandleCrash()
		le.config.Callbacks.OnStoppedLeading()
	}()
	// 尝试获取leader，失败返回
	if !le.acquire(ctx) {
		return // ctx signalled done
	}
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()
	// OnStartedLeading被设置为sched.Run，也就是只要当前node获得
	// 领导权，就开始运行调度器核心逻辑
	go le.config.Callbacks.OnStartedLeading(ctx)
	// renew是不断地获得领导权的节点轮训选举的状态，目的是保持自己的
	// 领导权
	le.renew(ctx)
}
```