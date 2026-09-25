# Consistency Models
What's Consistency ? Consistency describes what values a read is allowed to observe after writes occur in a distributed system.

Problem we are trying to solve here - How updated should the data be for the clients ?
You can try for the most consistent data, i.e. most recent write...but its not always possible with availability and partition tolerance tradeoffs that you have to make. Hence we decide on an approach and stick with it.

Things to learn here: 
Weak consistency
Eventual consistency
Strong consistency
Linearizability
Consistency vs replication
Synchronous vs asynchronous replication
Read-after-write consistency
Monotonic reads
Monotonic writes
Causal consistency
Sequential consistency
Session guarantees in depth
Formal mathematical definitions
Stale reads
Replica lag
Why eventual consistency can improve availability/latency
When strong consistency is actually necessary
2–3 concrete examples

## Approach 1 : Weak Consistency 
After a write, reads may or may not see it. A best effort approach is taken 

This approach is seen in systems such as memcached. Weak consistency works well in real time use cases such as VoIP, video chat, and realtime multiplayer games. For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

## Approach 2 : Eventual Consistency
After a write, reads will **eventually** see it (usually within miliseconds). Data is replicated asynchronously. 

This approach is seen in systems such as DNS and email. Eventual consistency works well in highly available systems

## Approach 3 : Strong Consistency
After a successful write, a subsequent read is guaranteed to observe that write.

This approach is seen in file systems and RDBMSes. Strong consistency works well in systems that need transactions.