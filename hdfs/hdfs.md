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

### How HDFS achieves its design goals
