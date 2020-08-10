Compare and Swap (CAS)
--- 

How it works?
--- 
CAS works as follows:<br/>
It first read the current value with the given one, if they are equal, swap in the new value and return true which means
the operation succeeded. Otherwise, it returns false to indicate the fail.
```
func cas(p *int, old int, new int) bool {
    if *p != old {
        return false
    }
    *p = new
    return true
}
```
The operation is atomic. The atomicity is usually guaranteed by hardware.

How to use?
---
```
func cas_incer(p *int, delta int) int {
    old = *p
    new = old + delta
    while ( !cas(p, old, new) ) {}
    return new
}
```

ABA Problem
---
Problem description:<br/>
Between the time the old value is read and the new value is attempted, some other processor changed the old value multiple
times while resulting one is equal to the old value but with a different meaning.  

Solution:<br/>
*Double length CAS (DCAS)*. E.g on a 32-bit system, use a 64-bit CAS. The second half serve as a version number counter.