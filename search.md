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
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/design-of-a-distributed-search  

1. Crawler
2. Indexer  
Index and Revert Index  
The indexer uses a distributed data processing system like MapReduce for parallel and distributed index construction.
4. Searcher
