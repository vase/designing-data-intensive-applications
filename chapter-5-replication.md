# Chapter 5 - Replication

Replication = keeping a copy of the same data on multiple machines that are connected via a network.

Why would you do this?
- Keep data geographically close to your users (geo-replication)
- Allow system to continue working even when some parts have failed. (high-availability)
- Scale out number of machines that can serve *read* queries (NOT write.)

Key assumption:
- Dataset is so small that each machine can hold the entirety of it. If one machine can't hold the entirety of your dataset, look at Chapter 7 - Partitioning (coming soon)

If data doesn't change over time, just make one copy to each machine and you're done. The key challenge lies in *handling changes to replicated data*

Types of replication:
- Synchronous replication
- Async replication

Every write needs to be processed by every replica, so how to distribute writes? A common solution to this is to designate one machine the _leader_, and the other machines the _followers_. Writes are only allowed to the leader, and the leader exposes a *replication log* that the followers read and use to update their own data store. This mode is used by MySQL, PostgreSQL, Oracale, MongoDB, and even message brokers like Kafka / RabbitMQ.

3 popular algorithms for replication changes:
- Single-leader
- Multi-leader
- Leaderless