# Distributed search
## Requirement
1. Functional requirements 
    Search: Users should get relevant content based on their search queries.

1. Non-functional requirements
    1. Availability: The system should be highly available to the users.
    1. Scalability: The system should have the ability to scale with the increasing amount of data. In other words, it should be able to index a large amount of data.
    1. Fast search on big data: The user should get the results quickly, no matter how much content they are searching.
    1. Reduced cost: The overall cost of building a search system should be less.
  
## Resource estimation
Storage/QPS/Bandwith/Number of servers

## High-level design
1. Crawler
2. Indexer  
Index and Revert Index  
The indexer uses a distributed data processing system like MapReduce for parallel and distributed index construction.
4. Searcher


<img width="691" alt="educative_distributed_search_high_level" src="https://github.com/toextendmylimits/system_design/assets/10056698/1f8cea07-dae7-476c-b463-754e8f90c1ab">

## Detailed discussion
### Distributed indexing and searching
The two most common techniques used for data partitioning in distributed indexing are these below:

1. Document partitioning: In document partitioning, all the documents collected by the web crawler are partitioned into subsets of documents. Each node then performs indexing on a subset of documents that are assigned to it.
1. Term partitioning: The dictionary of all terms is partitioned into subsets, with each subset residing at a single node. For example, a subset of documents is processed and indexed by a node containing the term “search.”

#### MapReuce
The MapReduce framework is implemented with the help of a cluster manager and a set of worker nodes categorized as Mappers and Reducers. As indicated by its name, MapReduce is composed of two phases:

The Map phase
The Reduction phase
Furthermore, the input to MapReduce is a number of partitions, or set of documents, whereas its output is an aggregated inverted index.

Let’s understand the purpose of the above components:

Cluster manager: The manager initiates the process by assigning a set of partitions to Mappers. Once the Mappers are done, the cluster manager assigns the output of Mappers to Reducers.

Mappers: This component extracts and filters terms from the partitions assigned to it by the cluster manager. These machines output inverted indexes in parallel, which serve as input to the Reducers.

Reducers: The reducer combines mappings for various terms to generate a summarized index.

The cluster manager ensures that all worker nodes are efficiently utilized in the cluster. The MapReduce is built to work under partial failures. If one node fails, it reschedules the work on another node.

Note that the Reducers cannot start as long as the Mappers are working. This means that the cluster manager can use the same node as a Mapper as well as a Reducer. 

### Data replication
