Linux context_switch() source code review
---

Takeaways
---
1. The two most important steps: *memory descriptor switch* and *register and stack switch*
2. What exactly do we need to switch? userspace memory mapping; userspace stack; kernel register; kernel stack.

Source code
---
- context_switch
```go
/*
 * context_switch - switch to the new MM and the new thread's register state.
 */
static __always_inline struct rq *
context_switch(struct rq *rq, struct task_struct *prev,
	       struct task_struct *next, struct rq_flags *rf)
{
	prepare_task_switch(rq, prev, next);

	/*
	 * For paravirt, this is coupled with an exit in switch_to to
	 * combine the page table reload and the switch backend into
	 * one hypercall.
	 */
	arch_start_context_switch(prev);

	/*
	 * kernel -> kernel   lazy + transfer active
	 *   user -> kernel   lazy + mmgrab() active
	 *
	 * kernel ->   user   switch + mmdrop() active
	 *   user ->   user   switch
	 */
	// next->mm == NULL, we are entering kernel
	// otherwise, we are entering user
	if (!next->mm) {                                
	    // this function tries to avoid tlb flush
	    // no need to switch userspace since we are
	    // entering kernel space
		enter_lazy_tlb(prev->active_mm, next);

        // we use the last process' active_mm
        // if the last one is a kernel , it will be NULL
		next->active_mm = prev->active_mm;
		// prev->mm != NULL means we come from a user process
		if (prev->mm)
		// increase reference count to this mm
			mmgrab(prev->active_mm);
		else
			prev->active_mm = NULL;
	} else {                                       
		// set rq->membarrier_state to next_mm->membarrier_state
		membarrier_switch_mm(rq, prev->active_mm, next->mm);
		/*
		 * sys_membarrier() requires an smp_mb() between setting
		 * rq->curr / membarrier_switch_mm() and returning to userspace.
		 *
		 * The below provides this either through switch_mm(), or in
		 * case 'prev->active_mm == next->mm' through
		 * finish_task_switch()'s mmdrop().
		 */
		switch_mm_irqs_off(prev->active_mm, next->mm, next);

        // from kernel
		if (!prev->mm) {                       
			rq->prev_mm = prev->active_mm;
			prev->active_mm = NULL;
		}
	}

	rq->clock_update_flags &= ~(RQCF_ACT_SKIP|RQCF_REQ_SKIP);

	prepare_lock_switch(rq, next, rf);

	/* Here we just switch the register state and the stack. */
	switch_to(prev, next, prev);
	// barrier makes sure that code below it 
	// will never run before code above it
	// i.e, disable instruction reordering
	barrier();

	return finish_task_switch(prev);
}
```

- prepare_task_switch
```go
/**
 * prepare_task_switch - prepare to switch tasks
 * @rq: the runqueue preparing to switch
 * @prev: the current task that is being switched out
 * @next: the task we are going to switch to.
 */
static inline void
prepare_task_switch(struct rq *rq, struct task_struct *prev,
		    struct task_struct *next)
{
	// kcov -> code coverage tool
	kcov_prepare_switch(prev);
	// schedule statistics
	sched_info_switch(rq, prev, next);
	// perf_event -> system performance profiling tool
	perf_event_task_sched_out(prev, next);
	// ?
	rseq_preempt(prev);
	// schedule out notifier
	fire_sched_out_preempt_notifiers(prev, next);
	// tell the process it's now on cpu
	prepare_task(next);
	// ?
	prepare_arch_switch(next);
}
```

