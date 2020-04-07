# Source Code analysis - GC()

## Takeaways
1. Golang GC is concurrent with user goroutine
2. Short STW period is needed to prepare background mark worker and root objects.
3. GC process:
        ① Concurrent Mark
        ② Mark Termination
        ③ Sweep
        ④ Sweep Termination

## Source code comment

```go
func GC() {

	// 首先等待上一轮GC的Mark、Mark termination和sweep termination都结束
	n := atomic.Load(&work.cycles)
	gcWaitOnMark(n)

	// 触发n+1轮GC
	gcStart(gcTrigger{kind: gcTriggerCycle, n: n + 1})

	// 等待这轮结束
	gcWaitOnMark(n + 1)

	// 确保span被清楚干净
	for atomic.Load(&work.cycles) == n+1 && sweepone() != ^uintptr(0) {
		sweep.nbgsweep++
		Gosched()
	}

	for atomic.Load(&work.cycles) == n+1 && atomic.Load(&mheap_.sweepers) != 0 {
		Gosched()
	}

	mp := acquirem()
	cycle := atomic.Load(&work.cycles)
	if cycle == n+1 || (gcphase == _GCmark && cycle == n+2) {
		mProf_PostSweep()
	}
	releasem(mp)
}
```

```go
func gcStart(trigger gcTrigger) {
	gcBgMarkStartWorkers()

	gcResetMarkState()

	...
	
	// 首先STW
	systemstack(stopTheWorldWithSema)
	// Finish sweep before we start concurrent scan.
	systemstack(func() {
		finishsweep_m()
	})
	
	...
	
	// STW后的主要工作
	// 1. 设置GC阶段标记为Mark
	// 2. 准备后台Mark
	// 3. Mark根对象
	setGCPhase(_GCmark)

	gcBgMarkPrepare() // Must happen before assist enable.
	gcMarkRootPrepare()

	// Mark all active tinyalloc blocks. Since we're
	// allocating from these, they need to be black like
	// other allocations. The alternative is to blacken
	// the tiny block on every allocation from it, which
	// would slow down the tiny allocator.
	gcMarkTinyAllocs()

	// At this point all Ps have enabled the write
	// barrier, thus maintaining the no white to
	// black invariant. Enable mutator assists to
	// put back-pressure on fast allocating
	// mutators.
	atomic.Store(&gcBlackenEnabled, 1)

    ...

	// Concurrent mark.
	// 上述工作做完，Start the world，进入并发Mark阶段（gc协程和应用协程并发）
	systemstack(func() {
		now = startTheWorldWithSema(trace.enabled)
		work.pauseNS += now - work.pauseStart
		work.tMark = now
	})
	// In STW mode, we could block the instant systemstack
	// returns, so don't do anything important here. Make sure we
	// block rather than returning to user code.
	if mode != gcBackgroundMode {
		Gosched()
	}

	semrelease(&work.startSema)
}
```