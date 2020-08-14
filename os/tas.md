Test and Set (TAS)
---

How it works?
---
Write 1 to the memory location and return the old value as an atomic operation.
```
func tas(p *int) int {
    old = *p
    *p = 1
    return old
}
```

How to use?
---
Test-and-set lock:
```
func lock(mux *int) {
    while ( tas(mux) == 1 ) {}
}
```

Performance evaluation:
---
Four metrics:<br/>
- lock-acquisition latency
- bus traffic
- fairness
- storage

