# Unique ID Generator
## Clarifying questions
1. Are these IDs numeric?
2. How long is the ID?
3. Does the ID get increased by time, for example, the ID generated in the evening is larger than ID generated in the morning on the same day?
4. What's the scale of the system, like how many IDs the system can generate in a second?
## Design
1 bit sign bit. Reserved for future purpose. Can be used to distinguish between signed and unsigned number
41 bit timestamp, roughly 2 trillion, can be used for 70 years. Each year has roughly 30 billion milliseconds.
5 bit data center
5 bit machines
12 bits sequence number
## Extra to discuss
1. Clock synchronization.
   In our design, we assume ID generation servers have the same clock. This assumption might not be true when a server is running on multiple cores. The same challenge exists in multi-machine scenarios. Solutions to clock synchronization are out of the scope of this course; however, it is important to understand the problem exists. Network Time Protocol is the most popular solution to this problem.
