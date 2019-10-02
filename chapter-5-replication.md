# Chapter 5 - Replication

Replication = keeping a copy of the same data on multiple machines that are connected via a network.

Why would you do this?
- Keep data geographically close to your users (geo-replication)
- Allow system to continue working even when some parts have failed. (high-availability)
- Scale out number of machines that can serve *read* queries (NOT write.)

Key assumption:
- Dataset is so small that each machine can hold the entirety of it. If one machine can't hold the entirety of your dataset, look at Chapter 7 - Partitioning (coming soon)

If data doesn't change over time, just make one copy to each machine and you're done. The key challenge lies in **handling changes to replicated data**

### Sync and Async replication

Types of replication:
- **Async Replication**: Leader doesn't wait for write success from follower before confirming success
- **Sync Replication**: Leader waits for write confirmation from at least a certain number of followers before confirming success.

*Note*: It is generally impractical for a replicated system to be fully synchronous, as any node having outage would bring the whole system down.

Some problems that arise with async replication:
- Reading data that has just been written may return stale data if the read is from a replica that hasn't been updated yet.
- Reading data implied from a fresh replica, then hitting a stale replica on second read. (See Fig 5-4, page 165)
- Causally-related reads are read in the wrong order from different replicas.

### Ways of replicating data

Every write needs to be processed by every replica, so how to distribute writes? A common solution to this is to designate one machine the _leader_, and the other machines the _followers_. Writes are only allowed to the leader, and the leader exposes a *replication log* that the followers read and use to update their own data store. This mode is used by MySQL, PostgreSQL, Oracale, MongoDB, and even message brokers like Kafka / RabbitMQ.

3 popular algorithms for replicating changes:
- Single-leader
- Multi-leader
- Leaderless

Key deliverable for replication: All replicas must end up with the **same** data on them as the leader and each other.


### Leader(ful?) Replication
Two types of failure:
- **Follower failure**: Catch-up once the follower recovers
- **Leader failure**: Failover, elect one of the remaining followers as new leader. This is generally very complicated and has huge margins for error. Common errors include not being able to detect when a leader has actually failed, _split brain_, dropping writes in case the newly elected leader has out-of-date data etc.

Replication logs have multiple types of implementation too:
- Statement-based replication = forward all writes to replicas
- Write-Ahead Log (WAL) = Leader forwards byte changes in the database as the update to followers
- Logical log = Leader sends a different log format from the storage engine to decouple storage and log.
- Trigger-based replication = Replication implemented in app code.

#### Multi-leader replication

##### Multi-datacenter operation
- Each DC would have it's own leader.
- Higher fault-tolerance, whole system doesn't go down just because one leader goes down. Other DCs can temporarily be secondaries to one DCs leader.
- Better write performance.

**Key Problem**: Write conflicts between leaders
Some solutions to the problem:
- Avoid conflict. If you can make sure all writes for a user go to a "home" datacenter, you don't need to handle conflict. But this is basically creating a separate DB system for each region in this case.
- Build convergence to a consistent state into the conflict resolution logic. Some ways for convergence include giving a unique ID to each write, and merge values together, or just throw away data that has less than the latest write ID.
- Use Conflict-Free Replicated Datatypes (CRDTs) (like Oprational transform)

#### Multi-leader write distribution
- Circular topology = leaders pass on writes to other leaders
- Star topology = Designated root passes on writes to all other leaders
- All-to-all topology (most common) = all leaders send writes to all other leaders.

### Leaderless Replication
- Most famous example of this is Amazon's DynamoDB. Other examples are Cassandra and Riak.
- Important concept is quorum. Writes are sent to all instances, and based on the configured quorum, the write is accepted or not.
