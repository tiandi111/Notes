Dynamic Scheduling
---

- Major limitation:

    Instruction is executed in-order, a stall stops all following instructions. Extra functional units are not used either.
    
- What we want?

    The instruction is executed as soon as operands are ready. Hence, we want out-of-order execution!
    
- Out-of-order execution
    - introduce WAR, WAW hazards
    - complicate exception handling