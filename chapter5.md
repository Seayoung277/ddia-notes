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

## Leaders and Followers

### Single-leader Replication
- One replica designated the leader. Leader sends out data change to all followers as replication log or change stream
- Only leader accepts write requests. All replicas accept query requests

### Synchronous VS Asynchronous Replication
- Difference: whether write request waits for replication to finish before responding
- Advantage of Sync: guaranteed to have up-to-date copy
- Disadvantage of Sync: block all writes and wait till replica is available
- In practice: one or a few followers are sync, the rest are async (semi-synchronous)

### Add New Followers
- Take a consistent snapshot of the leader's database
- Copy the snapshot to the new follower node
- New follower connects to the leader and requests all changes since the snapshot taken (catch up)

### Handle Node Outages

#### Follower Outages: Catch-up Recovery
- Same as bringing up a new follower

#### Leader outages: Failover
- Determining that the leader has failed
- Choosing a new leader
- Reconfiguring the system to use the new leader

#### Things Could Go Wrong During Failover
- New leader may not have all the writes from the old leader due to async replication, normally discarding un-replicated writes, which could cause problems if coordinating with other services
- Two nodes could both believe that they are the leader (split brain), which might cause data lost or corruption
- Choose the right timeout for determining a failover: long timeout means long recovery time, short timeout might cause false positive and unnecessary failover when lead is under heavy load