# Source code analysis — mallocgc() 

## Takeaways
About object allocation:<br>
1. small object is allocated from current per-P cache; particularly, a span with free object
2. large object is allocated from heap
3. if the span is full, the mcache's size-specific span will be refill from mcentral. See function refill() in Go/src/runtime/mcache.go.
4. Allocation path:<br/>
   (1) small object:
        mcache —a certain span is full—> refill from mcentral list -all span is full-> try sweep a span <br/>
       
   (2) large object          

About GC:<br/>
1. GC will be started when (1) a span is full (2)we are allocating a large object 
2. goroutine will not be preempted by GC while mallocing

## Source code comment
###1. mallocgc
function: mallocgc()<br/>
source code path: Go/src/runtime/malloc.go<br/>

```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
	... some code ...
	
    // assistG is the G to charge for this allocation, or nil if
    // GC is not currently active.
    // gcAssistBytes降到负值时，触发GC Mark，Best-effort形式
    // todo: then what is gcAssistBytes exactly?
    var assistG *g
    if gcBlackenEnabled != 0 {
        // Charge the current user G for this allocation.
        assistG = getg()
        if assistG.m.curg != nil {
            assistG = assistG.m.curg
        }
        // Charge the allocation against the G. We'll account
        // for internal fragmentation at the end of mallocgc.
        assistG.gcAssistBytes -= int64(size)

        if assistG.gcAssistBytes < 0 {
            // This G is in debt. Assist the GC to correct
            // this before allocating. This must happen
            // before disabling preemption.
            gcAssistAlloc(assistG)
        }
    }
	
    // Set mp.mallocing to keep from being preempted by GC.
    // 分配内存期间，当前goroutine不会被GC抢占
    mp := acquirem()
    if mp.mallocing != 0 {
        throw("malloc deadlock")
    }
    if mp.gsignal == getg() {
        throw("malloc during signal")
    }
    mp.mallocing = 1
    
    shouldhelpgc := false   // 指示是否进行gc
    dataSize := size        // 计算internal fragmentation
    c := gomcache()         // 获取当前Machine(OS Thread)的Cache
    var x unsafe.Pointer
    noscan := typ == nil || typ.kind&kindNoPointers != 0
    
    // 小对象
    if size <= maxSmallSize {
    	// 超小对象alloc
        if noscan && size < maxTinySize {
            off := c.tinyoffset
            // Align tiny pointer for required (conservative) alignment.
            // 超小对象size对齐
            if size&7 == 0 {
                off = round(off, 8)
            } else if size&3 == 0 {
                off = round(off, 4)
            } else if size&1 == 0 {
                off = round(off, 2)
            }
            // 当前Machine的Tiny Object Block有足够空间容纳该对象
            if off+size <= maxTinySize && c.tiny != 0 {
                // The object fits into existing tiny block.
                x = unsafe.Pointer(c.tiny + off)
                c.tinyoffset = off + size
                c.local_tinyallocs++
                mp.mallocing = 0    // indicate the termination of mallocing
                releasem(mp)        // release current machine
                return x
            }
            // Allocate a new maxTinySize block.
            span := c.alloc[tinySpanClass]
            v := nextFreeFast(span)
            if v == 0 {
            	//This line is important
            	//when the current tinySpanClass is full,
            	//a new gc will be started!
                v, _, shouldhelpgc = c.nextFree(tinySpanClass)
            }
            x = unsafe.Pointer(v)
            (*[2]uint64)(x)[0] = 0
            (*[2]uint64)(x)[1] = 0
            // See if we need to replace the existing tiny block with the new one
            // based on amount of remaining free space.
            if size < c.tinyoffset || c.tiny == 0 {
                c.tiny = uintptr(x)
                c.tinyoffset = size
            }
            size = maxTinySize
        // 小对象alloc
        } else {
            var sizeclass uint8
            if size <= smallSizeMax-8 {
                sizeclass = size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]
            } else {
                sizeclass = size_to_class128[(size-smallSizeMax+largeSizeDiv-1)/largeSizeDiv]
            }
            size = uintptr(class_to_size[sizeclass])
            spc := makeSpanClass(sizeclass, noscan)
            span := c.alloc[spc]
            v := nextFreeFast(span)
            if v == 0 {
                v, span, shouldhelpgc = c.nextFree(spc)
            }
            x = unsafe.Pointer(v)
            if needzero && span.needzero != 0 {
                memclrNoHeapPointers(unsafe.Pointer(v), size)
            }
        }
    // 大对象
    } else {
        var s *mspan
        // 每次alloc大对象，都会触发gc
        shouldhelpgc = true
        systemstack(func() {
            s = largeAlloc(size, needzero, noscan)
        })
        s.freeindex = 1
        s.allocCount = 1
        x = unsafe.Pointer(s.base())
        size = s.elemsize
    }
    
    // todo: understand the code block below
    var scanSize uintptr
    	if !noscan {
    		// If allocating a defer+arg block, now that we've picked a malloc size
    		// large enough to hold everything, cut the "asked for" size down to
    		// just the defer header, so that the GC bitmap will record the arg block
    		// as containing nothing at all (as if it were unused space at the end of
    		// a malloc block caused by size rounding).
    		// The defer arg areas are scanned as part of scanstack.
    		if typ == deferType {
    			dataSize = unsafe.Sizeof(_defer{})
    		}
    		heapBitsSetType(uintptr(x), size, dataSize, typ)
    		if dataSize > typ.size {
    			// Array allocation. If there are any
    			// pointers, GC has to scan to the last
    			// element.
    			if typ.ptrdata != 0 {
    				scanSize = dataSize - typ.size + typ.ptrdata
    			}
    		} else {
    			scanSize = typ.ptrdata
    		}
    		c.local_scan += scanSize
    	}
    
    	// Ensure that the stores above that initialize x to
    	// type-safe memory and set the heap bits occur before
    	// the caller can make x observable to the garbage
    	// collector. Otherwise, on weakly ordered machines,
    	// the garbage collector could follow a pointer to x,
    	// but see uninitialized memory or stale heap bits.
    	publicationBarrier()
    
    	// Allocate black during GC.
    	// All slots hold nil so no scanning is needed.
    	// This may be racing with GC so do it atomically if there can be
    	// a race marking the bit.
    	if gcphase != _GCoff {
    		gcmarknewobject(uintptr(x), size, scanSize)
    	}
    
    	... some code
    
    	mp.mallocing = 0
    	releasem(mp)
    
    	... some code ...
    
    	if assistG != nil {
    		// Account for internal fragmentation in the assist
    		// debt now that we know it.
    		assistG.gcAssistBytes -= int64(size - dataSize)
    	}
    
    	// 启动gc
    	if shouldhelpgc {
    		if t := (gcTrigger{kind: gcTriggerHeap}); t.test() {
    			gcStart(t)
    		}
    	}
    
    	return x
}
```

### Allocate from macache's span —— m.cache.nextFree()
```go
// nextFree returns the next free object from the cached span if one is available.
// Otherwise it refills the cache with a span with an available object and
// returns that object along with a flag indicating that this was a heavy
// weight allocation. If it is a heavy weight allocation the caller must
// determine whether a new GC cycle needs to be started or if the GC is active
// whether this goroutine needs to assist the GC.
//
// Must run in a non-preemptible context since otherwise the owner of
// c could change.
func (c *mcache) nextFree(spc spanClass) (v gclinkptr, s *mspan, shouldhelpgc bool) {
	s = c.alloc[spc]
	shouldhelpgc = false
	freeIndex := s.nextFreeIndex()
	// when the span is full, refill this spanClass from mcentral
	if freeIndex == s.nelems {
		
		... some code ...
		
		c.refill(spc)
		// gc is triggered when a span is full
		shouldhelpgc = true
		s = c.alloc[spc]

		freeIndex = s.nextFreeIndex()
	}

	... some code ...

	v = gclinkptr(freeIndex*s.elemsize + s.base())
	s.allocCount++
	
	... some code ...
	
	return
}
```

###Refill span from mcentral — mcache.refill()
```go
// refill acquires a new span of span class spc for c. This span will
// have at least one free object. The current span in c must be full.
//
// Must run in a non-preemptible context since otherwise the owner of
// c could change.
func (c *mcache) refill(spc spanClass) {
	// Return the current cached span to the central lists.
	s := c.alloc[spc]

	if uintptr(s.allocCount) != s.nelems {
		throw("refill of span with free space remaining")
	}
	if s != &emptymspan {
		// Mark this span as no longer cached.
		if s.sweepgen != mheap_.sweepgen+3 {
			throw("bad sweepgen in refill")
		}
		atomic.Store(&s.sweepgen, mheap_.sweepgen)
	}

	// Get a new cached span from the central lists.
	s = mheap_.central[spc].mcentral.cacheSpan()
	if s == nil {
		throw("out of memory")
	}

	if uintptr(s.allocCount) == s.nelems {
		throw("span has no free space")
	}

	// Indicate that this span is cached and prevent asynchronous
	// sweeping in the next sweep phase.
	s.sweepgen = mheap_.sweepgen + 3

	c.alloc[spc] = s
}
```

###Allocate span from mcentral — mcentral.cacheSpan()
```go
// Allocate a span to use in an mcache.
func (c *mcentral) cacheSpan() *mspan {
	// Deduct credit for this span allocation and sweep if necessary.
	spanBytes := uintptr(class_to_allocnpages[c.spanclass.sizeclass()]) * _PageSize
	deductSweepCredit(spanBytes, 0)

	lock(&c.lock)
	traceDone := false
	if trace.enabled {
		traceGCSweepStart()
	}
	sg := mheap_.sweepgen
retry:
	var s *mspan
	// iterate the nonempty span list
	for s = c.nonempty.first; s != nil; s = s.next {
		if s.sweepgen == sg-2 && atomic.Cas(&s.sweepgen, sg-2, sg-1) {
			c.nonempty.remove(s)
			c.empty.insertBack(s)
			unlock(&c.lock)
			s.sweep(true)
			goto havespan
		}
		if s.sweepgen == sg-1 {
			// the span is being swept by background sweeper, skip
			continue
		}
		// we have a nonempty span that does not require sweeping, allocate from it
		c.nonempty.remove(s)
		c.empty.insertBack(s)
		unlock(&c.lock)
		goto havespan
	}
    
	// nonempty span list is empty
	// try to sweep a span from empty list
	for s = c.empty.first; s != nil; s = s.next {
		if s.sweepgen == sg-2 && atomic.Cas(&s.sweepgen, sg-2, sg-1) {
			// we have an empty span that requires sweeping,
			// sweep it and see if we can free some space in it
			c.empty.remove(s)
			// swept spans are at the end of the list
			c.empty.insertBack(s)
			unlock(&c.lock)
			s.sweep(true)
			freeIndex := s.nextFreeIndex()
			if freeIndex != s.nelems {
				s.freeindex = freeIndex
				goto havespan
			}
			lock(&c.lock)
			// the span is still empty after sweep
			// it is already in the empty list, so just retry
			goto retry
		}
		if s.sweepgen == sg-1 {
			// the span is being swept by background sweeper, skip
			continue
		}
		// already swept empty span,
		// all subsequent ones must also be either swept or in process of sweeping
		break
	}
	if trace.enabled {
		traceGCSweepDone()
		traceDone = true
	}
	unlock(&c.lock)

	// Replenish central list if empty.
	s = c.grow()
	if s == nil {
		return nil
	}
	lock(&c.lock)
	c.empty.insertBack(s)
	unlock(&c.lock)

	// At this point s is a non-empty span, queued at the end of the empty list,
	// c is unlocked.
havespan:
	if trace.enabled && !traceDone {
		traceGCSweepDone()
	}
	n := int(s.nelems) - int(s.allocCount)
	if n == 0 || s.freeindex == s.nelems || uintptr(s.allocCount) == s.nelems {
		throw("span has no free objects")
	}
	// Assume all objects from this span will be allocated in the
	// mcache. If it gets uncached, we'll adjust this.
	atomic.Xadd64(&c.nmalloc, int64(n))
	usedBytes := uintptr(s.allocCount) * s.elemsize
	atomic.Xadd64(&memstats.heap_live, int64(spanBytes)-int64(usedBytes))
	if trace.enabled {
		// heap_live changed.
		traceHeapAlloc()
	}
	if gcBlackenEnabled != 0 {
		// heap_live changed.
		gcController.revise()
	}
	freeByteBase := s.freeindex &^ (64 - 1)
	whichByte := freeByteBase / 8
	// Init alloc bits cache.
	s.refillAllocCache(whichByte)

	// Adjust the allocCache so that s.freeindex corresponds to the low bit in
	// s.allocCache.
	s.allocCache >>= s.freeindex % 64

	return s
}
```