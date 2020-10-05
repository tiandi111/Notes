Appendix A — Instruction Set Architecture
---

## Category

The type of internal storage in the processor is the most basic differentiation.

- Stack
- Accumulator
- Register-Memory

    Can access memory directly from instructions.
        
- Register-Register (Load-Store)
    - Instructions are divided to two kinds: 
        - Memory Access: Load and Store between memory and register<br/>
        - ALU operations: which only occur between registers<br/>
        
GPRs: general purpose register.
        
## Memory Addressing

###1.Memory Interpretation
- Little Endian and Big Endian
- Alignment

###2.Addressing Mode
Why talk about addressing mode?<br/>
It has influences on：
- Instruction count
- Average clock cycle per instruction

Addressing mode:
- Register
- Immediate 
- Displacement 
       **Q:** why displacement size affect instruction size 
- Register Indirect
- Indexed
- Direct or Absolute
- Memory Indirect
- Autoincrement
- Autodecrement
- Scaled

## Type and Size of Operands

## Operations in Instruction Set
- Arithmetic and Logic
- Data transfer
- Control
- System
- Floating Point
- Decimal
- String
- Graphics

## Instructions for Control Flow

###1.Types:
- Conditional Branch
- Jump 
- Procedure Calls
- Procedure Returns

###2.Addressing Mode For Control Flow Instructions
The destination address of control flow instruction must always be specified.

- PC-Relative: supply a displacement that is added to program counter 
- Register Indirect: used in cases that the destination is unknown at compile time

###3.Conditional Branch Options
###4.Procedure Invocation Options
Procedure calls and returns include control transfer and possibly some state saving.
- Caller Saving
- Callee Saving

## Encoding an Instruction Set
How to encode instructions to binary format. The decision will affect:
- Compiled Program Size
- Implementation of Processor

####1.Instruction Format
```
General Instruction Format
+------------------------------+
| Opcode-Field | Address-Field |
+------------------------------+
```
- Variable: allow a large amount of addressing mode by using address specifier
    - smaller program size
    - best when there are many addressing modes and operations
- Fixed: fixed size for all instructions, combines the operation and the addressing mode into the opcode
    - best when there are few addressing mode and operations
- Hybrid: **Q**: Definition?

## The Role of Compiler

####1.The Structure of Recent Compilers
```
+-------------------------+
|  Front-end per langauge |
+-------------------------+
| High-level optimization | 
+-------------------------+
|     Global optimizer    |
+-------------------------+
|      Code generator     |
+-------------------------+
```

####2.Design of Compilers
- Design Goals:
    - First important goal: Correctness
    - Second important goal: Speed of compiled code
    - Other goals: fast compilation, debugging support, interoperability among languages
- Design Issues:
    - The complexity of writing a correct compiler is a major limitation on the amount of optimization that can be done
    - Multiple-pass structure helps reduce the complexity, but it also introduces "phase ordering problem"
- Optimization Styles:
    - High-level optimization
    - Local optimization: optimize code within a straight-line code fragment
    - Global optimization: extend local optimization across branches and introduces a set of transformations aimed at optimizing loops
    - Register allocation: associate register with operands (graph color problem)
    - Processor-dependent optimization: take advantage of specific architectural knowledge 
    
####3.The Impact of Compiler Technology on Architect's Decision
Two important questions:
- How are variable allocated and addressed
- How many registers are needed to allocate variables appropriately

####4.How the Architect Can Help the Compiler Writer
Some instruction set properties:
- Provide Regularity:
    
    Operations, data types and addressing mode should be orthogonal.
    
- Provide primitives, not solutions
- Simplify trade-offs among alternatives
- Provide instructions that bind the quantities known at compile time as constants

## RISC-V
RISC-V is a freely licensed open standard. It provides two basic instruction set, a 32-bit one and a 64-bit one, and a variety
of optional extensions to one of the basic instruction set.



