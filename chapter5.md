# Chapter 5: Replication

**Reasons of distributing data across multiple machines:**
Scalability, Availability, Latency

**Two common ways data is distributed across nodes:**
Replication, Partitioning

**Algorithms for replicating changes between nodes:**
- Single-leader/Active-passive/Master-slave
    - Sync, Async, Semi-sync
- Multi-leader
- Leader-less

**Automatic failover process:**
- Determining that the leader has failed
- Choosing a new leader
- Reconfiguring the system to use the new leader

## Leaders and Followers

### Single-leader Replication
- One replica designated the leader. Leader sends out data change to all followers as replication log or change stream
- Only leader accepts write requests. All replicas accept query requests

### Synchronous VS Asynchronous Replication
- Difference: whether write request waits for replication to finish before responding
- Advantage of Sync: guaranteed to have up-to-date copy
- Disadvantage of Sync: block all writes and wait till replica is available