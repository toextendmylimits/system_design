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

## Detailed Design

