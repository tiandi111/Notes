Context switch overhead
---

What to switch?
---
It really depends, but in linux, we have following things to switch:
- registers
- stack pointer
- program counter
- TLB
- page table

Where is the cost from?
---

- Direct cost
    - save and restore registers, stack pointer, program counter
    - flush TLB
    - load new page table if needed
    - running scheduler
    
- Indirect cost
    - all kinds of cache (page cache, buffer cache, L2 cache, etc.)
    - migrate the task among different CPU