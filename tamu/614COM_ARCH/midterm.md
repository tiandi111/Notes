Midterm
---

### Fundamentals

2. Computer Architecture
    - Instruction Set Architecture(ISA)
    - Functional Organization 
    - Hardware

3. Quantitative Principle
    - Processor Performance Equation
        ```
        CPU Time = IC * CPI * CCT
        ```
    - Amdahl's Law (quantify improvement)
        ```
        Speedup = 1 / (( 1-F ) + F / E)
        F is the fraction that can be enhanced, E is the speedup of the enhancement
        ```

### Instruction-level parallelism
1. ILP
    - Goal -> Improve CPI
    - How
        - Basic Pipelining (instruction overlap)
        - Explore More Parallelism
            - Hardware, dynamic
            - Software, static
    - Challenge
        - Structure Hazard
        - Control Hazard
        - Data Hazard
    
2. Overcome Structure Hazard
    - add more devices..

3. Overcome Control Hazard
    - Branch Predictor
        - Correlating Branch Predictor
        - Tournament Branch Predictor
        - Hybrid Branch Predictor
    - Speculation
        - Hardware Speculation
        - Software Speculation

4. Overcome Data Hazard
    - Hardware Forwarding
    - Instruction Scheduling
        - Dynamic Scheduling
            - tomasulo's algorithm
        - Static Scheduling
            - loop unrolling