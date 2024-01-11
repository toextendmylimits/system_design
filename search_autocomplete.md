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
