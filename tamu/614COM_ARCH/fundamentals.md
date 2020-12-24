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

## Trends in Power and Energy

### 1. Power and Energy: A Systems Perspective
- Maximum power
    - failed to meet max power requirement might lead to Malfunction
- Thermal design power (TDP)
    - What is it?
        - it is the amount of heat a component can generate, and the cooler must remove
        - it is not the peak power
        - it is not the actual average power
    - it often determines the cooling requirement
- Energy and energy efficiency
    - it is tied to a specific task
    - it is tied to execution time
    - thus, it is a better metric than power for comparing processor efficiency
    
### 2. Energy and Power within a Microprocessor
- Dynamic energy (caused by switching transistors)
    - Energy_dynamic ∝ Capacitive load * Voltage^2
    - Power_dynamic ∝ Capacitive load * Voltage^2 * Frequency switched
- Static energy
    - Energy_static ∝ Current_static * Voltage
- Techniques to improve energy efficiency
    - Clock gating
    - Power gating
    - 
    