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


## Read-after-write consistency - Session Guarantees for Weakly Consistent Replicated Data
Douglas B. Terry, Alan J. Demers, Karin Petersen, Mike J. Spreitzer, Marvin M. Theimer, and Brent B. Welch

The intent is to present individual applications with a view of the database that is consistent with their own actions, even if they read and write from various, potentially inconsistent servers. 
Four per-session guarantees are proposed to aid users and applications of weakly consistent replicated data: Read Your Writes, Monotonic Reads, Writes Follow Reads, and Monotonic Writes.
The guarantees can be layered on existing systems that employ a read-any/write-any replication scheme while retaining the principal benefits of such a scheme, namely high-availability, simplicity, scalability, and support for disconnected operation.


### Introduction
A session is an abstraction for the sequence of read and write operations performed during the execution of an application. Sessions are not intended to correspond to atomic transactions that ensure atomicity and serializability. Instead, the intent is to present individual applications with a view of the database that is consistent with their own actions, even if they read and write from various, potentially inconsistent servers.

Four gurantees : 
1. Read your writes - read operations reflect your previour write
2. Monotonic Reads - successive reads reflect a non-decreasing set of writes
3. Writes Follow Reads - Writes are propogated after reads on which they depend
4. Monotonic Writes - writes are propogated after the writes that logically precede them

What is the meaning of a "gurantee" here ? 
The storage system ensures above properties for each read a write operation belonging to a session or else, it informs the calling application that the guarantee(s) cannot be met. 


To ensure that the guarantees are met, the servers at which an operation can be performed must be restricted to a subset of available servers that are sufficiently up-to-date. Because enforcement of the guarantees restricts the set of servers that may be used within a session, requesting a guarantee can have an adverse impact on availability. Applications must make a trade-off between availability and consistency. For this reason, guarantees can be requested individually on a per-session basis. Requests for one or more of the guarantees within a session have no affect on the availability seen by applications that are using other sessions or on applications that require no guarantees.

### Read Your Writes
Users and applications find it particularly confusing if they update a database and then immediately read from the database only to discover that the update appears to be missing
Reads are restricted to copies of the database that include all previous Writes in this session. Specifically: RYW-guarantee: If Read R follows Write W in a session and R is performed at server S at time t, then W is included in DB(S,t).

### Monotonic Reads
The Monotonic Reads guarantee permits users to observe a database that is increasingly up-to-date over time. It ensures that Read operations are made only to database copies containing all Writes whose effects were seen by previous Reads within the session.
Basically, a read shall only be allowed from the database which is most updated by Writes, and whose effects were seen by the previous reads within the session. Let's take an example and try to breadown this definition on its two parts

A user's appointment calendar is stored on-line in a replicated database where it can be updated by both the user and automatic meeting schedulers. The user's calendar program periodically refreshes its display by reading all of today's calendar appointments from the database. If it accesses servers with inconsistent copies of the database, recently added (or deleted) meetings may appear to come and go. The MR-guarantee can effectively prevent this since it disallows access to copies of the database that are less current than the previously read copy. 


### Writes Follow Reads
The Writes Follow Reads guarantee ensures that traditional Write/Read dependencies are preserved in the ordering of Writes at all servers. That is, in every copy of the database, Writes made during the session are ordered after any Writes whose effects were seen by previous Reads in the session.
Imagine a shared bibliographic database to which users contribute entries describing published papers. Suppose that a user reads some entry, discovers that it is inaccurate, and then issues a Write to update the entry. For instance, the person might discover that the page numbers for a paper are wrong and then correct them with a Write such as "UPDATE bibdb SET pages = `45-53' WHERE bibid = `Jones93'." The WFR-guarantee can ensure that the new Write updates the previous bibliographic entry at all servers.
The WFR-guarantee, as defined, associates two constraints on Write operations. A constraint on Write order ensures that a Write properly follows previous relevant Writes in the global ordering that all database replicas will eventually reflect. A constraint on propagation ensures that all servers (and hence all clients) only see a Write after they have seen all the previous Writes on which it depends. Example 5 requires both of these properties. Some applications, however, may require only one of them. For such applications, systems may wish to provide relaxed variants of the Writes Follow Reads guarantee, one to guarantee how Writes are ordered and the other to guarantee how they propagate among servers:

The WFR-guarantee, as defined, associates two constraints on Write operations. A constraint on Write order ensures that a Write properly follows previous relevant Writes in the global ordering that all database replicas will eventually reflect. A constraint on propagation ensures that all servers (and hence all clients) only see a Write after they have seen all the previous Writes on which it depends. Above example requires both of these properties.

WFRO-guarantee: If Read R1 precedes Write W2 in a session and R1 is performed at server S1 at time t1, then WriteOrder(W1,W2) for any W1 in RelevantWrites(S1,t1,R1).
WFRP-guarantee: If Read R1 precedes Write W2 in a session and R1 is performed at server S1 at time t1, then, for any server S2, if W2 is in DB(S2) then any W1 in RelevantWrites(S1,t1,R1) is also in DB(S2).

### Monotonic Writes
The Monotonic Writes guarantee says that Writes must follow previous Writes within the session. In other words, a Write is only incorporated into a server's database copy if the copy includes all previous session Writes; the Write is ordered after the previous Writes.

The MW-guarantee could be used by a text editor when editing replicated files to ensure that if the user saves version N of the file and later saves version N+1 then version N+1 will replace version N at all servers. In particular, it avoids the situation in which version N is written to some server and version N+1 to a different server and the versions get propagated such that version N is applied after N+1.

### Providing the guarantees
A server must be willing to return information about the unique identifier (WID) assigned to a new Write, the set of WIDs for Writes that are relevant to a given Read, and the set of WIDs for all Writes in its database.

The burden of providing the guarantees lies primarily with the session manager through which all of a session's Read and Write operations are serialized. The session manager can be considered a component of the client stub that mediates communication with available servers. For each session, it maintains two sets of WIDs:

read-set = set of WIDs for the Writes that are relevant to session Reads
write-set = set of WIDs for those Writes performed in the session


## Causal Consistency - Bolt-on Causal Consistency

### Introduction
A user updates data in one region, and the change propagates asynchronously to the others. This improves availability and reduces coordination, but replicas may temporarily disagree.

There are three important concepts:
- Availability: Can the system continue responding to requests?
- Eventual consistency: Will replicas eventually converge if updates stop?
- Causal consistency: Will users observe an event's causes before observing its effects?
Eventual consistency alone does not guarantee the third property. That is the gap this paper addresses.

Event A — Original post
Sally posts: "My son is missing!"
[*Sally learns he is safe and edits her post*]

Event B — Correction
"False alarm! He's safe."
[*A friend reads the correction and responds*]

Event C — Reply
"What a relief!"

Suppose another user sees the reply but not the correction. They might see the friend expressing relief while still believing the child is missing. That is a causal consistency violation: the user observed an effect without its causal dependency.

### What does bolt-on means ? 
The authors propose separating database responsibilities into two layers.

1. Application / Client : Issues reads and writes

2. Bolt-on shim layer : Tracks dependencies, maintains metadata, and controls which writes become visible

3. Eventually consistent database : Handles storage, replication, durability, availability, and eventual convergence

Think of the shim as a consistency-enforcing middleware layer.
The underlying database doesn't need to understand the application's causal relationships. The shim maintains that information and mediates what clients can observe.

### Problem of Overwritten Histories
This is one of the paper's most important technical contributions.

Consider two writes to the same key:
- y1 is an earlier value.
- y2 overwrites y1.
Suppose another write, x1, depends on y1.

The shim needs to know that x1 depends on y1 to preserve causality. But an eventually consistent database may expose only the latest value, y2. If the shim hasn't observed y1 before it gets overwritten, the information it needs may disappear.

The authors call this the overwritten histories problem. 

Why is this difficult?
- Waiting indefinitely for y1 may make the system unavailable.
- Ignoring the missing dependency may violate causal consistency.
- Keeping every historical version forever can create substantial storage overhead.
The paper develops a mechanism that addresses these competing requirements without requiring the underlying store to retain every write.

### Solution
The authors introduce a declarative concept called a causal cut.

A causal cut is a set of writes that is safe to expose together: whenever an included write depends on an earlier write, the set contains an appropriate version of that dependency.
Consider this : 
Write A
*Creates a record*

Write B
*Depends on A*

Write C
*Depends on B and indirectly on A*

If the shim wants to expose C, it must ensure the causal dependencies are covered. It cannot simply expose C because the underlying database happens to return it.

To do this, the shim:
1. Attaches dependency metadata to writes.
2. Keeps a local store containing a causally valid set of writes.
3. Checks whether a write's dependencies are already covered.
4. Fetches dependency information when needed.
5. Makes writes visible only when the causal requirements are satisfied.

The paper describes both an asynchronous approach and a more pessimistic approach that checks dependencies on the read path.

### Approach A: Asynchronous dependency resolution
- the standard bolt-on causal consistency design
The key idea is: never make a read wait for dependency resolution.

Suppose the local store contains A, but the database has A, B, and C.
Client requests C *Read immediately*

1. The shim returns the value currently in its local store.
Local store: A
C isn't applied yet, so the read may return an older value or no value.

2. The shim adds the relevant key to a background-check queue.
Background resolver:
- Fetches the latest available write.
- Checks its dependencies.
- Fetches missing dependencies where possible.
- Applies the writes in a causally valid order.

3. A later read can observe the newly applied data.
The background resolver maintains a set of keys called to_check. A read requests a check for its key, and the resolver polls the underlying database because the database doesn't necessarily notify the shim when new writes arrive.

**What if a dependency is missing?**
Suppose the resolver finds C, which depends on B, but B is not in the local store. It attempts to fetch B. If B depends on A, the resolver must also ensure that A is covered.
The resolver builds a tentative set of writes and applies them only if that set, together with the local store, satisfies the causal-cut condition. This prevents it from exposing a partial dependency chain. 

**Advantages and disadvantages**
- Very fast reads: They can be served from local memory.
- Good availability: Reads don't need to wait for the underlying database.
- Potentially stale results: A recent write may exist in the database but not yet in the local store.
- Poor first-read visibility: The paper notes that the first read of a key can return null if the resolver hasn't already been checking that key.

### Approach B: Pessimistic dependency resolution
The pessimistic approach moves the same dependency-resolution work into the read's critical path. Writes still follow the same basic process.

Using the same example:
Client requests C *Resolve before returning*

1. Read C from the database : The shim fetches the latest available version of C.

2. Check dependencies : C depends on B. Is B covered by the local store or the writes already being resolved?

3. Fetch missing dependencies : If B is missing, read it from the database and recursively check its dependencies, such as A.

4. Apply and return : If a valid causal cut can be formed, apply the writes and return C. Otherwise, the newly fetched writes aren't applied.

**Advantages and disadvantages**
- Better visibility: It tries to incorporate newer data before answering.
- Higher read latency: Each missing dependency can trigger another database request.
- Lower throughput: The read path now performs network and dependency-resolution work.
- Long dependency chains hurt: A read can trigger multiple sequential fetches before it completes.

### Practical Implementation
The authors' single-node experiments on Cassandra found a significant performance difference between the two designs. 

Asynchronous bolt-on : 16.3K–29.2K Peak operations per second across the evaluated datasets
Pessimistic bolt-on : 8.8K–15.0K Peak operations per second across the evaluated datasets

For the Metafilter workload, the pessimistic design performed an average of 1.32 underlying database reads per frontend read, while the asynchronous design performed none on the synchronous read path. Pessimistic resolution also produced 18% fewer null responses in that workload.