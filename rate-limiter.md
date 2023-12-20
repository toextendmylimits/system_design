# Rate Limiter

## Requirement
### Functional requirement
1. To limit the number of requests a client can send to an API within a time window.
1. To make the limit of requests per window configurable.
1. To make sure that the client gets a message (error or notification) whenever the defined threshold is crossed within a single server or combination of servers.
### Non-unctional requirement
1. Availability: Essentially, the rate limiter protects our system. Therefore, it should be highly available.
1. Low latency: Because all API requests pass through the rate limiter, it should work with a minimum latency without affecting the user experience.
1. Scalability: Our design should be highly scalable. It should be able to rate limit an increasing number of clients’ requests over time.
### Where to place the rate limiter
1. On client side
2. On server side
3. As middleware

## High-level design
Refer to https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/design-of-a-rate-limiter
https://bytebytego.com/courses/system-design-interview/design-a-rate-limiter

1. Race condition
 Locks are the most obvious solution for solving race condition. However, locks will significantly slow down the system. Two strategies are commonly used to solve the problem: Lua script [13] and sorted sets data structure in Redis

3. Synchronization issue  
 Centralized data store like Redis

## Algorithms
1. Token bucket algorithm
2. Leaking bucket algorithm
 When a request arrives, the system checks if the queue is full. If it is not full, the request is added to the queue.  
 Otherwise, the request is dropped.  
 Requests are pulled from the queue and processed at regular intervals.
3. Fixed window counter algorithm
4. Sliding window log algorithm
