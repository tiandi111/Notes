Computer Organization and Desgin
---
Chapter 4 | The Processor
---

## 4.5 An Overview of Pipeline

### 1. Intro
[Pipelining] Pipelining is an implementation technique in which multiple instructions overlapped
in execution. This improves the throughput, not the latency.

- single-cycle model
    - every instruction takes one clock cycle
    - clock cycle must accommodate the slowest instruction
- pipelining
    - every stage takes one clock cycle
    - clock cycle must accommodate the slowest stage

The equality holds when pipe stages are perfectly balanced.
```
    SPEEDUPpipe <= Number of pipe stages
```

Instruction stages:
- IF: instruction fetch
- ID: instruction Decode
- EX: execution (ALU)
- MEM: memory access
- WB: write back data from register to memory
```
+------+------+------+-------+------+
|  IF  |  ID  |  EX  |  MEM  |  WB  |
+------+------+------+-------+------+
```

### 2. Pipeline Hazard
- Structural Hazard

    The hardware cannot support the combination of instructions that we want to execute in the same
    clock cycle.

- Data Hazard

    The data that is needed to execute the instruction has not yet ready.
    
    - forwarding(bypassing): Retrieving the missing data element from internal buffers earlier rather
    than waiting for it. 
    - Load-use data hazard: the data being loaded by a load instruction has not yet become available when
    it is needed by another instruction.
    - pipeline stall: a stall initiated in order to resolve the hazard.

- Control Hazard(Branch Hazard)

    The instruction fetched is not the expected one. 
    
    - stall
    - prediction
        
        Always predict the branch will be taken or not taken.
        
    - delayed branch
        
        By reordering instructions or execute a non-sense instruction after branch instruction.
        
### 3. Pipelined Datapath and Control

### 4. Data Hazard: Forwarding vs Stalling

- Forwarding
    - When?
        - an instruction tries to use a register in its EX stage that an earlier instruction tends to
        write in its WB stage
        - from EX/MEM pipeline register or MEM/WB pipeline register
    - How?
        - forwarding unit + multiplexor
        - 
- Stall
    - When？
        - read a register following a load instruction that writes the same register
    - How？
        - hazard detection unit
        - insert nop 

### 5. Control Hazards

- Assume branch untaken
- Reducing the delay of branches
- Dynamic Branch Prediction
    - branch prediction buffer(branch history table)
    