A comprehensive survey for scheduling techniques in cloud computing
---

1.Resource provisioning
---
- On-demand Provisioning按需分配
- Advanced Reservation提前预留
- Spot Instance现货实例

2.Resource Scheduling
---
- Static Scheduling:
    - advanced knowledge is needed
    - easy to implement
    - bad performance, not optimized QoS
    - FIFO, RR, SJF etc
   
- Dynamic Scheduling:
    - need to continuously monitor nodes
    - more efficient and accurate
    - need continuously monitoring; hard to implement 
    - dynamic round robin, heterogeneous earliest finish time 
      (HEFT), clustering based heterogeneous with duplication 
      (CBHD), weighted least connection (WLC), particle swarm 
      optimization (PSO), ant colony optimization (ACO) etc. 
      
- Online Scheduling:
    - dynamic round robin, heterogeneous
      earliest finish time (HEFT), clustering based heterogeneous
       with duplication (CBHD), weighted least connection (WLC), 
       particle swarm optimization (PSO), ant colony optimization 
       (ACO) etc. 
       
- Offline Scheduling:
    - Max-Min, Min-Min

- Preemptive and Non-Preemptive Scheduling

3.Classification of scheduling scheme
---

- Heuristic scheduling algorithm:
    - HEFT
    
