ECEN676 Advanced Computer Architecture Summary
---

Part Memory Hierarchy: 
---
- Cache 
    - Layout
        - PIPT
        - PIVT
        - VIPT
        - VIVT
    - Optimization
        - larger block size
            - reduce compulsory miss: exploit spatial locality 
            - increase conflict miss: reduce number of blocks
            - increase miss penalty
        - higher associativity
            - reduce conflict miss
            - may increase hit time
        - larger cache size
            - reduce conflict miss
            - increase miss penalty
            - higher cost and power
        
    - 