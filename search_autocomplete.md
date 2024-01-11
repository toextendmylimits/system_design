# Design a system that can suggest the rop N searches to the usser based on what is typed in the search box
## Reqiurement
### Clarifying questions
1.  Is the matching only supported at the beginning of a search query or in the middle as well?
2.  How many autocomplete suggestions should the system return?
3.  How does the system know which 5 suggestions to return?
4.  Do we allow capitalization and special characters?
5.  How many users use the product?

### Functional Requirement
1. The system should suggest the top N frequent and relevant terms to the user based on what is typed in the search box

### Non-functional requirement
1. Low latency  
The system should show all the suggested queries in real time after a user types. The latency shouldn’t exceed 200 ms. A study suggests that the average time between two keystrokes is 160 milliseconds. So the latency should be more
than 160 ms, but it shouldn't too high as otherwise a suggestion might be stale and less useful.  
2. Highly available  
The system should be available to provide suggestions despite the failure of one or more of its components.
3. Scalable  
The system should support the ever-increasing number of users over time.

### Resource estimation Storage/QPS/Servers
Let's assume there are 2 billion unique queries a day, and each query has 30 characters, and each character is of 2 Bytes, then the storage for a day is:  
2 billion * 30 charcters * 2 Bytes = 120GB per day, storage per year is probably 40 TB.

QPS = 2 billion / 86400 = 2000 query per sercond

## High-level Design
<img width="862" alt="search_autocomplete_high_level" src="https://github.com/toextendmylimits/system_design/assets/10056698/c0880692-d069-440b-8ca5-5f2bdd378a0d">

Our proposed system should do the following:  
1. Provide suggestions based on the search history of the user.  
1. Store all the new and trending queries in the database to include them in the list of suggestions.  

Key components:
1. Suggestion service
TO obtains the top ten suggestions from the cache, Redis, and returns them as a response to the client.
2. Assembler
 An assembler collects the user searches, applies some analytics to rank the searches, and stores them in a NoSQL database that’s distributed across several nodes.

Furthermore, we also need load balancers to distribute the incoming requests evenly. We also add application servers as entry points for clients so that they can forward requests to the appropriate microservices. These web servers encapsulate the internal system architecture and provide other services, such as authentication, monitoring,

API
1. get suggestion  
getSuggestion(query)
2. Add trending quereis to database
addToDatabase(query)

## Trie data structure
The issue we’re attempting to tackle is that we have many strings that we need to store in a way that allows users to search for them using any prefix. Our service suggests the next words that match the provided prefix. Let’s suppose our database contains the phrases UNITED, UNIQUE, UNIVERSAL, and UNIVERSITY. Our system should suggest “UNIVERSAL” and “UNIVERSITY” when the user types “UNIV.”

A trie is a tree-like data structure for storing phrases, with each tree node storing a character in the phrase in order. If we needed to store UNITED, UNIQUE, UNIVERSAL, and UNIVERSITY in the trie, it would look like this:

### Track the top searches
Since our system keeps track of the top searches and returns the top suggestion, we store the number of times each term is searched in the trie node. Let’s say that a user searches for UNITED 15 times, UNIQUE 20 times, UNIVERSAL 21 times, and UNIVERSITY 25 times. 

### Trie partitioning
We aim to design a system like Google that we can use to handle billions of queries every second. One server isn’t sufficient to handle such an enormous amount of requests. In addition to this, storing all the prefixes in a single trie isn’t a viable option for the system’s availability, scalability, and durability. A good solution is to split the trie into multiple tries for a better user experience.

Let’s assume that the trie is split into two parts, and each part has a replica for durability purposes. All the prefixes starting from “A” to “M” are stored on Server/01, and the replica is stored on Server/02. Similarly, all the prefixes starting from “N” to “Z” are stored on Server/03, and the replica is stored on Server/04. 

### Update the trie
Billions of searches every day give us hundreds of thousands of queries per second. Therefore, the process of updating a trie for every query is highly resource intensive and time-consuming and could hamper our read requests. This issue can be resolved by updating the trie offline after a specific interval. To update the trie offline, we log the queries and their frequency in a hash table and aggregate the data at regular intervals. After a specific amount of time, the trie is updated with the aggregated information. After the update of the trie, all the previous entries are deleted from the hash table.

One way to update the trie to have one primary copy and several secondary copies of the trie. While the main copy is used to answer the queries, we may update the secondary copy. We may also make the secondary our main copy once the upgrade is complete. We can then upgrade our previous primary, which will then be able to serve the traffic as well.
