# Availability vs Consistency
I would consider you done with 0.3 when you can explain all of this without notes:

Fundamentals
- What availability means
- What consistency means
- What a network partition is
- Why distributed systems experience partitions
- Difference between partition and node failure
CAP
- State CAP in your own words
- Explain why partition is the critical condition
- Derive the C/A tradeoff from a two-node example
- Explain CP
- Explain AP
- Explain why “pick any two” is misleading
Reasoning
- Explain what happens to reads during a partition
- Explain what happens to writes during a partition
- Explain stale data
- Explain conflicting writes
- Explain why coordination can reduce availability
- Explain why accepting independent writes can hurt consistency

only pick two out of : Consistency, Availability, Partition Tolerance.

- **Consistency** - Every read receives the most recent write or an error
- **Availability** - Every request receives a response, without guarantee that it contains the most recent version of the information
- **Partition Tolerance** - The system continues to operate despite arbitrary partitioning due to network failures

*Networks aren't reliable, so you'll need to support partition tolerance. You'll need to make a software tradeoff between consistency and availability.*

Your first question would be, why not all three ?
consider the following case :
You have two servers A & B both are read servers and responsible for sending data back to client in case of a relevant request. There is a third server C, called write server. 

        A  ------------   B
        |                 |
        |<-------C------->|

**Case 1** : Everything in this world works fine. 
Can we ensure **consistency** ? Yes since both servers are able to communicate to the write server and keep themselves updated, we can rest assured the C will be able to update A and B

Can we ensure **availability** ? Yes, since both server are available. 

Can we ensure **partition tolerance** ? Not yet tested. No partition has occurred.

**Case 2** : Real world scenario - partition happens (b/w AC and BC)
can we ensure **consistency** ? Yes, iff A and B both return an error saying its not updated with the latest information. 

can we ensure **availability** ? Yes, iff A and B both return stale data!

This is where we see consistency and availability choosing different paths. 

*Now partititon happens almost always since network is un-reliable (rule of thumb) Hence we ask you to choose b/w C & A .*

##  Why distributed systems experience partitions
Becasue they primarily talk to each other over *the network*. Since the network is a bunch of cables lying deep below the ocean with sharks constantly chewing on the cables (or the ship anchors dragging across them), they are susceptible to failures. 

More formally, distributed systems experience network partitions because their nodes communicate over networks, which cannot guarantee uninterrupted, reliable communication. Failures such as cable damage, router failures, packet loss, congestion, and configuration errors can prevent some nodes from exchanging messages. The nodes may remain operational even though communication between them is disrupted.

## Difference between partition and node failure
Partition occurs when the network b/w the nodes fails to communicate the information. Node failures occur when the node itself stops responding. Node failure can be handled by taking proper availability measures and creating a fail-safe architecture. Partition failure is something completly different. 
More formally, A node failure occurs when a machine or process becomes unable to perform its role or respond to requests. A network partition occurs when otherwise-operational nodes cannot reliably communicate with one another. A node may be healthy locally but appear unreachable to another node because of a network failure. Distributed systems must account for both.

## State CAP in your own words
In my own words - CAP theorem talks about consistency, availabilty and partition tolerance in distributed systems. It emphasizes on choosing b/w C & A while taking proper precautionary measures about P. 
In better words - CAP theorem states that a distributed system cannot guarantee both consistency and availability while also tolerating a network partition. During a partition, it must sacrifice either consistency or availability for affected operations. When there is no partition, consistency and availability can coexist.

## CP & AP
once you decide that P has to be taken care of... You are left with two choices. C & A and tbh your system has already decided which one would eventually suit your requirements. 

CP : Waiting for a response from the partitioned node might result in a timeout error. It is a good choice if your business needs require atomic reads and writes.

AP :  Responses return the most readily available version of the data available on any node, which might not be the latest. Writes might take some time to propagate when the partition is resolved. AP is a good choice if the business needs to allow for eventual consistency or when the system needs to continue working despite external errors.

## Explain why “pick any two” is misleading
Your system should strive for both A & C when P is upright. 

## Explain what happens to reads during a partition
in CP model -may reject or block a read if it cannot guarantee the required consistency. But not every read necessarily fails.For example, a read from a node that can still safely establish the latest value may succeed.

in AP model - may return locally available data without guaranteeing that it reflects the latest write. It could be stale, but it could also happen to be current.


## Explain what happens to writes during a partition
in CP model - may reject writes that cannot be safely coordinated, while still accepting writes in parts of the system that can maintain the required consistency.
in AP model - may accept writes independently on reachable replicas, allowing temporary divergence. Once communication is restored, it needs a reconciliation strategy to resolve differences. 

## Explain conflicting writes
Conflicting writes occur when multiple independent writes affect the same piece of data and cannot be reconciled automatically without a resolution policy. In a partitioned system, two replicas may accept different updates to the same key. Once communication is restored, the system needs a strategy to reconcile them.
A conflict typically arises when two writes are made independently, often concurrently, and the system needs to determine how to reconcile them.

Example:
Initial state:x = 0
-----Network partition occurs------
Client 1 → Node A: x = 10
Client 2 → Node B: x = 20
A accepts x = 10
B accepts x = 20
---------Partition heals----------

How should the system resolve x?
Now you have independently accepted updates that may conflict.Possible resolution strategies include:
- Last-write-wins.
- Application-specific conflict resolution.
- Merging updates.
- Rejecting one of the conflicting operations.

## Explain why coordination can reduce availability
Coordination can reduce availability because operations may need agreement from a leader, quorum, or other replicas before completing. If a partition prevents that coordination, the system may have to reject or delay requests rather than risk violating its consistency guarantee.

## Explain why accepting independent writes can hurt consistency
Consistency requires communication b/w all the nodes accepting the writes, because in the case of network partitions, non-communication may lead to in-coherent data b/w the nodes. Consider the above example of A & B being two write nodes (instead of read nodes). In case of partition b/w A & B, a client making a write req might only be received by one of the nodes, for eg : A is the one accepting the req, will not be communicated to B. Now A has the most up-to-date, most-recent info while B has stale data which it potentially doesn't even know. 

# Challenge
Scenario: 3-node distributed database
You have three replicas: A, B, and C.
A can communicate with B.
B can communicate with C.
A cannot communicate with C.
All three machines are healthy.
A client writes x = 10 to A.

Question: Is this necessarily a network partition? And can a CP system still accept that write? Explain your reasoning. 

A network partition is a communication failure between nodes, but the absence of a direct connection doesn't necessarily mean there's no indirect communication path.
A CP system can still accept a write if it can safely establish the required quorum or coordination guarantee. For example, in a three-node consensus group, if A is the valid leader and can communicate with B, A and B may form a majority quorum and commit the write without C. C can catch up later.
However, if A cannot establish the required quorum, it may have to reject or delay the write to preserve consistency.Therefore, the outcome depends on the replication topology and consensus protocol, not simply on whether every replica is reachable.