Chapter 1 — Fundamentals
---

## Classes of Parallelism
Parallelism at multiple levels is now the driving force of computer design across all four classes of computers, with
energy and cost being the primary constraints.

Classes of parallelism in applications:
- Data-level parallelism(DLP)

    Arises because there are many data items that can be operated on at the same time.

- Task-level parallelism(TLP)

    Arises because tasks of work are created that can operate independently and largely in parallel.
    
Classes of architectural parallelism:
- Instruction-level parallelism 

    Exploits data-level parallelism at modest levels with compiler help using ideas like pipelining and at medium levels using ideas like speculative execution.

- Vector architectures / Graphic processor units(GPUs)

    Exploit data-level parallelism by applying a single instruction to a collection of data in parallel.

- Thread-level parallelism

    Exploits either data-level parallelism or task-level parallelism in a tightly coupled hardware model that allows for interaction between parallel threads.

- Request-level parallelism

    Exploits parallelism among largely decoupled tasks specified by the programmer or the operating system.

Flynn's taxonomy:
- SISD
- SIMD
- MISD
- MIMD

## Designing Computer Architecture
---

###1. Instruction Set Architecture
Aspects:
- Class of ISA
    - stack
    - accumulator
    - register-memory
    - register-register(load-store)
- Memory Addressing
    - byte addressing
    - alignment
- Addressing Mode
- Types and Sizes of operands
- Operations
    - data transfer
    - arithmetic logical
    - control
    - floating point
- Control Flow Instruction
    - conditional branches
    - unconditional jumps
    - procedure calls
    - procedure returns
- Encoding an ISA
    - fixed length
    - variable length
    - hybrid
    
###2. Genuine Computer Architecture: Designing the Organization and Hardware to Meet Goals and Functional Requirements
The implementation of a computer:
- Organization(microarchitecture)
- Hardware


    