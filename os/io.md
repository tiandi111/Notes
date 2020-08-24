I/O System
---

### Principles of I/O Hardware

- Categories
    - block devices 
        - store info in fixed-size blocks
        - random access
    - character devices
        - character stream
    - network
    - clocks and timers
    - others
    
- Device Controllers
    
    The electronic part (chips, pcbs, etc.) that controls the device.
    
- Memory-Mapped I/O
    
    Map I/O ports onto memory space so that the CPU can access them as if
    it is accessing main memory.

- Direct Memory Access
    
    DMA allows certain hardware subsystems to access main memory independent of
    CPU so that the CPU does not need to polling those I/O hardware, i.e. Programmed I/O. 

### Principles of I/O Software

- Goals of the I/O software
    - device independence
    - uniform naming
        - e.g., in Unix all disks can be mounted in file system
    - error handling
    - synchronous vs asynchronous
    - blocking vs non-blocking
    - buffering
    
- Programmed I/O
    
    To have the CPU do all the work. Usually incorporate polling or busy waiting.
    
- Interrupt-Driven I/O

- I/O Using DMA

### I/O Models

- Blocking I/O
```
        request
+---+  -------->  +---+
| A |          |  | B |
+---+  <-------+  +---+
      result ready!
```

- Non-Blocking I/O
```
         request
+---+   -------->    +---+
|   |           |    |   |
|   |   <-------+    |   |
|   |   response     |   |
|   |                |   |
|   |     check      |   |
|   |   -------->    |   |
|   |   <--------    |   |
| A |   not ready    | B |
|   |       .        |   |
|   |       .        |   |
|   |       .        |   |
|   |     check      |   |
|   |   -------->    |   |
|   |   <--------    |   |
+---+ result ready!  +---+
```

- I/O Multiplexing
```
          request
+---+     -------->         +---+
|   |             |A ready? |   |
|   |             |B ready? |   |
|   |             |C ready? |   |
|   |     <-------+         |   |
|   |   resource A aval!    |   |
|   |                       |   |
| A |       f(A)            | B |
|   |     -------->         |   |
|   |             |         |   |
|   |             |         |   |
|   |    <--------+         |   |
+---+   result ready！      +---+
```

- Signal-Driven I/O
```
          request
+---+    -------->    +---+
|   |    <--------    |   |
|   |    response     |   |
|   |                 |   |
|   |      signal     |   |
| A |    <--------    | B |
|   |     callback    |   |
|   |    -------->    |   |
|   |            |    |   |
|   |    <--------    |   |
+---+  result ready!  +---+
```

- Asynchronous I/O
```
        request
+---+   -------->   +---+
|   |   <--------   |   |
| A |   response    | B |
|   |               |   |
|   |   <--------   |   |
+---+ result ready! +---+
```

    
### I/O Software Layer

#### Scheduling

The OS implements scheduling by maintaining a wait queue for each device. 
The I/O scheduler will rearrange the order of the requests in the wait queue to improve the overall performance
of the I/O subsystem.


