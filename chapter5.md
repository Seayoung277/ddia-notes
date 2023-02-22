# Chapter 5: Replication

Reasons of distributing data across multiple machines: Scalability, Availability, Latency

Two common ways data is distributed across nodes: Replication, Partitioning

Algorithms for replicating changes between nodes:
- Single-leader/Active-passive/Master-slave
    - Sync, Async, Semi-sync
- Multi-leader
- Leaderless

Automatic failover process:
- Determining that the leader has failed
- Choosing a new leader
- Reconfiguring the system to use the new leader