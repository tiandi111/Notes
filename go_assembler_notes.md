# Go Assembler notes

## Day 1： runtime.systemstack(fn func())

1. Why call on system stack?<br/>

User stacks and system stacks:

Every non-dead G has a *user stack* associated with it, which is what
user Go code executes on. User stacks start small (e.g., 2K) and grow
or shrink dynamically.

Every M has a *system stack* associated with it (also known as the M's
"g0" stack because it's implemented as a stub G) and, on Unix
platforms, a *signal stack* (also known as the M's "gsignal" stack).
System and signal stacks cannot grow, but are large enough to execute
runtime and cgo code (8K in a pure Go binary; system-allocated in a
cgo binary).

Runtime code often temporarily switches to the system stack using
`systemstack`, `mcall`, or `asmcgocall` to perform tasks that must not
be preempted, that must not grow the user stack, or that switch user
goroutines. Code running on the system stack is implicitly
non-preemptible and the garbage collector does not scan system stacks.
While running on the system stack, the current user stack is not used
for execution.

2. MOVQ fn+0(FP), DI<br/>
-> move the first argument of this function, i.e fn,  to register DI<br/>
-> FP：a pseudo-register, frame pointer: arguments and locals.
    
3. get_tls(CX)<br/>
-> #define	get_tls(r)	MOVQ TLS, r<br/>
-> 

4. 