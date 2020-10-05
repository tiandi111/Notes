Appendix B — Memory Hierarchy
---

## Cache Performance Review
```
CPU Execution time = (CPU clock cycles + Memory stall cycles) * CCT
Memory stall cycles = Number of misses * Miss penalty
Number of misses = IC * (Memory Accesses * Miss rate / Instructions)
```

## Cache Design Issues
### 1. Block placement
- Direct Mapped
    
    Block can only be placed at on position, usually (block no.) mod (cache size in blocks).
    
- Fully Associative

    Block can be placed anywhere.

- Set Associative

    Block is direct mapped to a set and can be placed anywhere inside the set.
    
    N-way set: the number of blocks in a set is N.
    
### 2. Block Identification

- Block offset: which byte inside the block
- Index: locate the set
- Tag: Higher bits of the real block address
- Valid bit: if the data is valid
```
+-----------------------------+
|    Block Address  |  Block  |
+-------------------|         |
|  Tag  |   Index   |  offset |
+-----------------------------+
```
Tags are usually compared parallel.


### 3. Block Replacement
- Random
- LRU and Apporx LRU
- FIFO
- No choice(in direct mapped  mode)

### 4. Write Strategy
- Write Through
    - Pros and Cons
        - simple 
        - in favour of data coherency
- Write Back
    - Pros and Cons
        - performance efficient (multiple writes, only one go to the memory)
        - power efficient
    - Write stall
    - On write miss