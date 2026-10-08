Things to learn here: 
Weak consistency
Eventual consistency
Strong consistency
Linearizability 
Consistency vs replication
Synchronous vs asynchronous replication
Read-after-write consistency : https://www.cs.cornell.edu/courses/cs734/2000FA/cached%20papers/SessionGuaranteesPDIS_1.html

Monotonic reads
Monotonic writes
Causal consistency - https://www.bailis.org/papers/bolton-sigmod2013.pdf?utm_source=chatgpt.com
Sequential consistency - MIT 6.5840 / 6.824 lecture material
Formal mathematical definitions - https://cs.brown.edu/people/mph/HerlihyW90/p463-herlihy.pdf?utm_source=chatgpt.com

Stale reads
Replica lag
Why eventual consistency can improve availability/latency
When strong consistency is actually necessary
2–3 concrete examples

# Consistency Models
What's Consistency ? Consistency describes what values a read is allowed to observe after writes occur in a distributed system.

Problem we are trying to solve here - How updated should the data be for the clients ?
You can try for the most consistent data, i.e. most recent write...but its not always possible with availability and partition tolerance tradeoffs that you have to make. Hence we decide on an approach and stick with it.

## Approach 1 : Weak Consistency 
After a write, reads may or may not see it. A best effort approach is taken 

This approach is seen in systems such as memcached. Weak consistency works well in real time use cases such as VoIP, video chat, and realtime multiplayer games. For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

## Approach 2 : Eventual Consistency
After a write, reads will **eventually** see it (usually within miliseconds). Data is replicated asynchronously. 

This approach is seen in systems such as DNS and email. Eventual consistency works well in highly available systems

## Approach 3 : Strong Consistency
After a successful write, a subsequent read is guaranteed to observe that write.

This approach is seen in file systems and RDBMSes. Strong consistency works well in systems that need transactions.

# Linearizability
**Basic Idea:** to make a system appear as if there was only one copy of the data, and all operations on it  atomic. With this guarantee, even though there may be multiple replicas in reality, the application does not need to worry about them. Linearizability is a recency guarantee on reads and writes of a register (an object). It  doesn’t group operations together into transactions, so it does not prevent problems such as write skew unless you take additional measures such as materializing conflicts

## Uses of linearizability

1. Locking and Leader election
2. Constraints and uniqueness guarantees
3. Cross-channel timing dependencies

Single leader systems are generally linearizable. In distributed systems, you can use Consensus algorithms to make them linearizable up to a large extent

Concept                      	|               Concern
Linearizability	                |   Real-time ordering of individual operations
Serializability	                |   Transactions appear to execute serially
Strict serializability	        |   Serializability + real-time ordering


# Consistency vs replication

Replication answers: "How many copies of the data do we maintain, and how do we propagate changes?"
Consistency answers: "What can clients observe when those copies differ?"

# Replication Methods and Their Relevant Consistency Guarantees

## Synchronous Replication
The write isn't acknowledged until the required replicas have persisted it.
Advantages:
- stronger durability
- potentially stronger consistency
- less chance of losing acknowledged writes
Disadvantages:
- higher write latency
- availability problems if replicas are unavailable

## Asynchronous Replication
Primary acknowledges immediately.
Replication happens afterward.
Advantages:
- lower latency
- higher availability
- better geographic performance
Disadvantage:
- replica lag
- stale reads

