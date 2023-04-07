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

## Single-leader Replication

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

### Replication Log Implementation

#### Statement-based Replication
- Leader logs every statement and send to followers to execute
- Problems:
    - Nondeterministic function calls will result in different values on each replica (e.g. NOW() or RAND())
    - Statements using auto-incrementing column or depending on existing data in the database mush be executed in exactly same order on each replica
    - Statements having side effects (e.g. triggers, stored procedures, user-defined functions) may result in different side effects on each replica

#### Write-ahead Log (WAL) Shipping
- Most databases uses append-only sequence of bytes containing all writes to the database.
- Besides writing the log to disk, send it to followers as well
- Problems:
    - WAL describes data on very low level, replication is closely coupled to the storage engine.
    - Changing storage format is difficult, not able to run different versions of the database software
    - Meaning zero-downtime upgrade is difficult

#### Logical (Row-based) Log Replication
- A logical log for a relational database is a sequence of records describing writes to database tables at the granularity of a row
- Decoupled from storage engine internals, easier to be kept backward compatible.
- Easier for external applications, e.g. data warehouse or custom indexes and caches, to parse (change data capture)

#### Trigger-based Replication
- Application layer replication
- Use trigger to register custom application code to execute when data changes
- Flexible but greater overheads

## Replication Lag

- Replication can only be asynchronous in practice
- Results in eventual consistency

### Read after Write

- While read from followers, new contents haven't been replicated from leader to the follower
- Complication arises when user is accessing the service from different devices, i.e. cross-device read-after-write consistency.

#### How to achieve read-after-write consistency:
- When read something might have modified by the user, read from leader; otherwise read from follower. 
    - For example, always read the user's own profile from the leader, but any other users' profile from followers
- Use last updated time to determine whether to query from leader or follower
- Monitor the replication lag of followers and avoid querying from followers with replication lag greater than an threshold
- Client provides a timestamp telling the system to only execute the query from either leader or followers with revision greater than the timestamp

### Monotonic Reads

- When users read from different replicas, acquired contents might move back in time due to replication lag.
- Monotonic Reads is weaker than strong consistency but stronger than eventual consistency

#### How to achieve Monotonic Reads

- Make sure users only read from the same replica, or only read from replicas with newer data than the previous replica

### Consistent Prefix Reads

- If a sequence of writes happens in a certain order, anyone reading those writes will see them appear in the same order
- This is a particular problem in partitioned databases. For those databases different partitions operate independently, and there's no global ordering of writes.

#### How to achieve Consistent Prefix Reads

- Make sure any writes that are causally related to each other are written to the same partition

### Fundamental Solution of Replication Lag

- Transactions are the way for databases to provide strong guarantees so that application don't need to worry about the subtle replication issues.
- Single-node transactions is easy, but distributed databases rarely supports transactions

## Multi-Leader Replication

### Use Cases

#### Multi-datacenter Operation

- Each datacenter has it's own leader
- Each leader acts as a follower to the other leaders
- Each datacenter's leader replicates its changes to the leaders in other datacenters
- Within each datacenter, regular leader-follower replication is used
- Benefits:
    - Every write can be processed in the local data center and replicated asynchronously to other datacenters. inter-datacenter network delay is hidden from users
    - Each datacenter can continue operation independently of the others, replication catches up when failed datacenter comes back online
- Multi-leader replication is often considered dangerous territory that should be avoided if possible

#### Clients with Offline Operation

- Application needs to continue to work while it is disconnected from internet
- Each device has a local database that acts as the leader

#### Collaborative Editing

- Each user editing the same doc first persists the changes to their local replica, then synced to the server and other editors replica asynchronously

### Handling Write Conflicts

- Makes the conflict detection synchronous, i.e. wait for the write to be replicated to all replicas before telling the user that the write was successful
- Avoid conflict, i.e. if the application can ensure that all writes for a particular record go through the same leader, then conflict cannot occur.
- Resolve conflicts in a convergent way, which means all replicas must arrive a the same final value when all changes have been replicated.
    - Give each write a unique ID, pick the write with the highest ID as the winner and throw away other writes
    - Give each replica a unique ID and let writes that originated at a higher-numbered replica always take precedence over others
    - Merge the values together, e.g. order them alphabetically and then concatenate them
    - Record the conflict in an explicit data structure that preserves all information and write application code that resolves the conflict later
- Custom conflict resolution logic
    - **On write**: As soon as the database system detects a conflict it calls the conflict handler
    - **On Read**: When conflict is detected, all conflicting writes are stored. When data is read, conflict handler is called
    