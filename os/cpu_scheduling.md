CPU Scheduling
---

### Criteria

- CPU utilization
- Throughput
- Turnaround time 
    - the time from the submission of the process to the completion of the process
- Waiting time
    - the time the process spend in the waiting queue
- Response time
    - the time from the submission of the process to the first response produced
    
### Scheduling Algorithm

- FCFS (First Come First Server)
    - easy
    - low performance

- SJF (Shortest Job First)
    - how to predict the running time of a process? 
   
- Time-slice
    - how to decide time slice length?

- Priority 
    - starvation is a problem

- Multilevel queue
    - queue has priority, a queue can use CPU only when all higher-prior queues are empty
    - queue can also have time slice

- Multilevel feedback queue
    - allow process to move between queues, upgrade or demote
    - each queue can have its own scheduling algorithm

### Scheduling in Linux 

#### 1. Scheduling timing

- the running task terminates
- the running task sleeps
- a new task created or sleeping task wakes up
- the running task's timeslcie is over

#### 2. Current scheduler design - multilevel feedback queue 

1. Components
    - scheduling queue is called "scheduling class" in linux, different classes have their
    own scheduing algorithm
    - scheduling classes are scheduled in the order of priority
    - task can migrate between CPUs, scheduling policies and scheduling classes
    
2. Top-level scheduling algorithm

Starting from the highest priority class, pick the next runnable task to run on CPU.

```
again:
    for_each_class( class ) {
        p = class -> pick_next_task( rq, prev, rt);
        if (p) {
            if (unlikely(p==RETRY_TASK))
                goto again;
            return p
        }
    }
```

3. Scheduling classes and policies
    - STOP
        - No policy
    - DEADLINE
        - SCHED_DEADLINE
    - REAL TIME
        - SCHED_FIFO
        - SCHED_RR
    - CFS (Completely fair scheduler)
        - SCHED_NORMAL
        - SCHED_BATCH
        - SCHED_IDLE
    - IDLE
        - No policy