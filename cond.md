Condition variable in golang
---

What is condition variable?
---
Condition variable is a mechanism for inter-goroutine communication.
It parks a goroutine waiting for some event to happen and later wake
up it to continue from the parking point once the event has happened.

Why condition variable?
---
A goroutine can wait for some event to happen by two ways:<br/>
(1) poll the event status<br/>
(2) park itself, yield the cpu and wait for notification<br>

In the first way, we are burning extra cpu cycles but once the event
happens, we can immediately continue on the goroutine. 

In the second way, we yield the cpu to other goroutine to better use it.
But once the event happens, the goroutine is not immediately continued 
but put it to a run queue waiting for scheduling. Park and
reschedule the goroutine requires two goroutine context-switch.

How condition variable works?
---
**- wait**<br/>
Get the waiter ticket number and wait for notification to it.
```go
func (c *Cond) Wait() {
	c.checker.check()
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}
```
Atomically increase waiter ticker number and return it.
```go
func notifyListAdd(l *notifyList) uint32 {
	return atomic.Xadd(&l.wait, 1) - 1
}
```
The 2 main steps in notifyListWait is:<br/>
(1) enqueue the current goroutine to cond.notifyList. s is of type sudog,
it is a representation of g in wait list(one g can have multiple sudog since
it can be in multiple wait list)<br/>
(2) park the current goroutine and do a schedule.
```go
func notifyListWait(l *notifyList, t uint32) {
	lock(&l.lock)
    ...
	// get sudug representation
	s := acquireSudog()
	s.g = getg()
	s.ticket = t
	...
	// enqueue itself
	if l.tail == nil {
		l.head = s
	} else {
		l.tail.next = s
	}
	l.tail = s
	// park the current goroutine
	goparkunlock(&l.lock, waitReasonSyncCondWait, traceEvGoBlockCond, 3)
	...
	releaseSudog(s)
}
```
**- notify**
When notify, the target g is found and readied. It will be put into the current
run queue waiting for scheduled.
```go
func notifyListNotifyOne(l *notifyList) {
	...
	// Update the next notify ticket number.
	atomic.Store(&l.notify, t+1)

	// Try to find the g that needs to be notified.
	// If it hasn't made it to the list yet we won't find it,
	// but it won't park itself once it sees the new notify number.
	//
	// This scan looks linear but essentially always stops quickly.
	// Because g's queue separately from taking numbers,
	// there may be minor reorderings in the list, but we
	// expect the g we're looking for to be near the front.
	// The g has others in front of it on the list only to the
	// extent that it lost the race, so the iteration will not
	// be too long. This applies even when the g is missing:
	// it hasn't yet gotten to sleep and has lost the race to
	// the (few) other g's that we find on the list.
	for p, s := (*sudog)(nil), l.head; s != nil; p, s = s, s.next {
		if s.ticket == t {
			n := s.next
			if p != nil {
				p.next = n
			} else {
				l.head = n
			}
			if n == nil {
				l.tail = p
			}
			unlock(&l.lock)
			s.next = nil
			readyWithTime(s, 4)
			return
		}
	}
	unlock(&l.lock)
}
```