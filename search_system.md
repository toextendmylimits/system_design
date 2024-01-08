# A search system
## Requirement
### Clarifying questions
1. What is the key funcationality? Is it to take some text input, a search query, from the user and returns the relevant content in a few seconds or less?
1. How large is the data to be searched?
1. How many daily active users does the system have?

### Funciontional requirements
To take some text input from the user and returns relevant content in a few seconds

### Non-functional requirements
1. Availability
2. Scalability
The system should have the ability to scale with the increasing amount of data.
3. Fast search on big data:
The user should get the results quickly, no matter how much content they are searching.
4. Reduced cost

## Design
### Key components
1. Crawler 
To fetch contents and create documents
1. Indexer  
Builds a searchable indexer 
1. Searcher  
responds to search queries by running the search query on the index created by the indexer.

What does indexer do?
1. To create index for each document
Running search queries on billions of documents that are document-level indexed will be a slow process, which may take many minutes, or even hours.
1. To create inverted index
It's a hashmap-like data structure. Each entry is a mapping of a unique word and the documents that this word appear, the frequency and locations it appear each document.

Indexing on a centralized system
We can create indexing on a centralized system but it has issues of being single point of failure and could be hard to scale as well. So a better option is to create index in a distributed environment.

### Design diagram

<img width="662" alt="search_system_design" src="https://github.com/toextendmylimits/system_design/assets/10056698/abc1d96d-ee39-42e5-9f3b-8e80c0125b20">

### How Indexer works in distributed environment in details
#### Data partitioning
1. Document partitioning:
In document partitioning, all the documents collected by the web crawler are partitioned into subsets of documents. Each node then performs indexing on a subset of documents that are assigned to it.
1. Term partitioning:
The dictionary of all terms is partitioned into subsets, with each subset residing at a single node. For example, a subset of documents is processed and indexed by a node containing the term “search.”
#### 
