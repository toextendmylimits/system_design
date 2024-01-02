# Unique ID Generator
## Clarifying questions
1. Are these IDs numeric?
2. How long is the ID?
3. Does the ID get increased by time, for example, the ID generated in the evening is larger than ID generated in the morning on the same day?  Are the IDs ordered by date?  
4. What's the scale of the system, like how many IDs the system can generate in a second?
## Design
1. SQL database auto-increment feature
   1. Single database server
   1. Have k databases servers, each one generate id with auto increment of k. For example, if k is 2, server 1 generate id 1, 3, 5, while server 2 generated id 2, 4, 6
      Pros:
      Easy to implement.
      
      Disadvantages:
      1. IDs don't go up with time across multiple servers
      2. Hard to scale with multiple data centers
      3. It doesn't scale well when a server is added or removed
  
1. UUID  
   UUID is a 128-bit number used to identify information in computer systems. UUID has a very low probability of getting collusion.
   
   Pros:
   1. Generating UUID is simple. No coordination between servers is needed so there will not be any synchronization issues.
   1. The system is easy to scale because each web server is responsible for generating IDs they consume. ID generator can easily scale with web servers.

   Cons: 
   1. IDs are 128 bits long, but our requirement is 64 bits.  
   1. IDs do not go up with time.
   1. IDs could be non-numeric.
      
1. Twitter snowflake
   1 bit sign bit. Reserved for future purpose. Can be used to distinguish between signed and unsigned number  
   41 bit timestamp, roughly 2 trillion, can be used for 70 years. Each year has roughly 30 billion milliseconds.  
   5 bit data center  
   5 bit machines  
   12 bits sequence number

   Pros#
   1. Twitter Snowflake uses the time stamp as the first component. Therefore, they’re time sortable. The ID generator is highly available as well.

   Cons
   1. IDs generated in a dead period are a problem. The dead period is when no request for generating an ID is made to the server. These IDs will be wasted since they take up identifier space. The unique range possible will deplete earlier than expected and create gaps in our global set of user IDs.
      
## Extra to discuss
1. Clock synchronization.
   In our design, we assume ID generation servers have the same clock. This assumption might not be true when a server is running on multiple cores. The same challenge exists in multi-machine scenarios. Solutions to clock synchronization are out of the scope of this course; however, it is important to understand the problem exists. Network Time Protocol is the most popular solution to this problem.
