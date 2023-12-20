# Design a distributed key value store
## Requirement
get/put

1. Ability to store big data.
2. High availability: The system responds quickly, even during failures.
3. High scalability: The system can be scaled to support large data set.
4. Low latency
5. Tunable consistency.

## Key system components
### Data partitition  
  There are two challenges while partitioning the data:
  1.Distribute data across multiple servers evenly.
  1.Minimize data movement when nodes are added or removed.  
   
   Consistent hashing
   
### Data replication  
   To achieve high availability and reliability, data must be replicated asynchronously over N servers, where N is a configurable parameter.  

   With virtual nodes, the first N nodes on the ring may be owned by fewer than N physical servers. To avoid this issue, we only choose unique servers while performing the clockwise walk logic.

### Consistency  
   Since data is replicated at multiple nodes, it must be synchronized across replicas. Quorum consensus can guarantee consistency for both read and write operation

   N = The number of replicas  
  W = A write quorum of size W. For a write operation to be considered as successful, write operation must be acknowledged from W replicas.  
  R = A read quorum of size R. For a read operation to be considered as successful, read operation must wait for responses from at least R replicas.

  If W + R > N, strong consistency is guaranteed because there must be at least one overlapping node that has the latest data to ensure consistency.
  
1. Consistency model  
   1. Strong consistency: any read operation returns a value corresponding to the result of the most updated write data item. A client never sees out-of-date data.
   1. Weak consistency: subsequent read operations may not see the most updated value.
   1. Eventual consistency: this is a specific form of weak consistency. Given enough time, all updates are propagated, and all replicas are consistent.

2. Inconsistency resolution
   Data Versioning

### Handling failures
  1. Failure detection

   In a distributed system, it is insufficient to believe that a server is down because another server says so. Usually, it requires at least two independent sources of information to mark a server down.  
     
     A better solution is to use decentralized failure detection methods like gossip protocol. Gossip protocol works as follows:  
     Each node maintains a node membership list, which contains member IDs and heartbeat counters.  
     Each node periodically increments its heartbeat counter.  
     Each node periodically sends heartbeats to a set of random nodes, which in turn propagate to another set of nodes.  
     Once nodes receive heartbeats, membership list is updated to the latest info.  
      
     If the heartbeat has not increased for more than predefined periods, the member is considered as offline.  
    
