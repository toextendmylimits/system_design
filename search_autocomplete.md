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

![Uploading search_autocomplete_high_level_design.png…]()


Key components:
1. Suggestion service
Provide suggestions based on user's search history and what's the relevant trending search query
2. Assembler
Store all the new and trending queries in the database to include them in the list of suggestions.


