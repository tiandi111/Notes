Notes on 《HDFS Architecture Guide》
---

### Design goal
- Highly fault-tolerant 
    - detection of faults
    - quick recovery
    - automatic recovery
- Streaming
    - batch processing than interactive use
    - high throughput than low latency
- Support large files
    - high aggregate data bandwidth ?
    - scale to hundreds of nodes ?
- Write-once-read-many
    - strictly one writer at a time
- Good data locality
    - moving computation is cheaper than moving data
- Support multiple platforms and devices

### Architecture
- NameNode
    - one per cluster
    - manage namespace
    - manage block-to-DataNode mappings
    - manage healthy DataNode list by monitoring heartbeat
- DataNode
    - serve clients' write and read requests
    - perform block creation, deletion and replication (instructed by NameNode)
    
### Replication
- File structure
    - files are stored as a sequence of blocks
    - blocks are distributed across the whole cluster
    - NameNode manages the mappings from blocks to DataNodes
    - block size and replication factor are configurable for each file
- Replication
    - Replication placement
        - goals and some facts
            - to optimize data reliability, availability and network bandwidth
            - network bandwidth within a rack is larger than that across different racks 
            - 
        - rack-aware placement policy
        - 
    - Replica selection
    - Safe mode
### How HDFS achieves its design goals
