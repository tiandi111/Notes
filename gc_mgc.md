# Source Code analysis - GC()

## Takeaways
1. Golang GC is concurrent with user goroutine
2. Short STW period is needed to prepare background mark worker and root objects.
3. GC process:
        ① Concurrent Mark <br/>
        ② Mark Termination <br/>
        ③ Sweep <br/>
        ④ Sweep Termination <br/>
4. GC trigger conditions: <br/>
        ① According to GcPercent, pseudo code: <br/>
        ```
         if heap_mem >= (1+gcpercent) * heap_mem_after_last_gc {
                        gcStart()
                    }
        ```
        ② Every two minutes <br/>
        ③ According to GC cycle <br/>

## Source code comment
### 1. GC mian process
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

### 2. gcStart
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
	// 2. 准备后台Mark Worker
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

### 3. test trigger condition
```go
func (t gcTrigger) test() bool {
	if !memstats.enablegc || panicking != 0 {
		return false
	}
	if t.kind == gcTriggerAlways {
		return true
	}
	if gcphase != _GCoff {
		return false
	}
	switch t.kind {
	// 堆内存触发条件，gc_trigger == 0.95 * gcpercent
	case gcTriggerHeap:
		// Non-atomic access to heap_live for performance. If
		// we are going to trigger on this, this thread just
		// atomically wrote heap_live anyway and we'll see our
		// own write.
		return memstats.heap_live >= memstats.gc_trigger
	// 时间条件：每隔2分钟会触发一次
	case gcTriggerTime:
		if gcpercent < 0 {
			return false
		}
		lastgc := int64(atomic.Load64(&memstats.last_gc_nanotime))
		return lastgc != 0 && t.now-lastgc > forcegcperiod
	// Trigger循环触发条件
	case gcTriggerCycle:
		// t.n > work.cycles, but accounting for wraparound.
		return int32(t.n-work.cycles) > 0
	}
	return true
}
```
每隔2分钟强制GC
```go
// forcegcperiod is the maximum time in nanoseconds between garbage
// collections. If we go this long without a garbage collection, one
// is forced to run.
//
// This is a variable for testing purposes. It normally doesn't change.
var forcegcperiod int64 = 2 * 60 * 1e9
```
gc_trigger的计算
```go
trigger = uint64(float64(memstats.heap_marked) * (1 + triggerRatio))
minTrigger := heapminimum
if trigger < minTrigger {
    trigger = minTrigger
}
memstats.gc_trigger = trigger
```

### 4. Background sweep
When a goroutine is started, a corresponding bgsweep() goroutine will also be started.
```go
func bgsweep(c chan int) {
	sweep.g = getg()

	lock(&sweep.lock)
	sweep.parked = true
	c <- 1
	goparkunlock(&sweep.lock, waitReasonGCSweepWait, traceEvGoBlock, 1)

	for {
		for sweepone() != ^uintptr(0) {
			sweep.nbgsweep++
			Gosched()
		}
		for freeSomeWbufs(true) {
			Gosched()
		}
		lock(&sweep.lock)
		if !isSweepDone() {
			// This can happen if a GC runs between
			// gosweepone returning ^0 above
			// and the lock being acquired.
			unlock(&sweep.lock)
			continue
		}
		sweep.parked = true
		goparkunlock(&sweep.lock, waitReasonGCSweepWait, traceEvGoBlock, 1)
	}
}
```

### 5. Force GC Helper
When process is started, a forcegchelper is started to trigger GC every 2 minutes
```go
func init() {
	go forcegchelper()
}

func forcegchelper() {
	forcegc.g = getg()
	for {
		lock(&forcegc.lock)
		if forcegc.idle != 0 {
			throw("forcegc: phase error")
		}
		atomic.Store(&forcegc.idle, 1)
		goparkunlock(&forcegc.lock, waitReasonForceGGIdle, traceEvGoBlock, 1)
		// this goroutine is explicitly resumed by sysmon
		if debug.gctrace > 0 {
			println("GC forced")
		}
		// Time-triggered, fully concurrent.
		gcStart(gcTrigger{kind: gcTriggerTime, now: nanotime()})
	}
}
```