# Rate Limiter

## Requirement
### Clarifying questions:
1. What is the key functionality? Is it to limit the number of requests a client can send to an API within a time window?
2. Is the limit configurable?
3. Does the rate limiter throttle API requests based on IP, the user ID, or other properties?
4. Do we need to inform users when their requests are throttled?
5. What is the scale of the system? Is it built for a startup or a big company with a large user base?
### Functional requirement
1. To limit the number of requests a client can send to an API within a time window.
1. To make the limit of requests per window configurable.
1. To make sure that the client gets a message (error or notification) whenever the defined threshold is crossed within a single server or combination of servers.
### Non-unctional requirement
1. Availability: Essentially, the rate limiter protects our system. Therefore, it should be highly available.
1. Low latency: Because all API requests pass through the rate limiter, it should work with a minimum latency without affecting the user experience.
1. Scalability: Our design should be highly scalable. It should be able to rate limit an increasing number of clients’ requests over time.

## Types of throttling
1. hard throttling
 So, whenever a request exceeds the limit, it is discarded.
3. soft throttling
Under soft throttling, the number of requests can exceed the predefined limit by a certain percentage. 
5. dynmaic throttling
### Where to place the rate limiter
1. On client side
It is easy to place the rate limiter on the client side. However, this strategy is not safe because it can easily be tampered with by malicious activity. 
3. On server side
In this approach, a server receives a request that is passed through the rate limiter that resides on the server.
5. As middleware

## High-level design
Refer to https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/design-of-a-rate-limiter
https://bytebytego.com/courses/system-design-interview/design-a-rate-limiter

![Uploading Screen Shot 2024-01-06 at 4.04.56 pm.png…]

1. Race condition
 Locks are the most obvious solution for solving race condition. However, locks will significantly slow down the system. Two strategies are commonly used to solve the problem: Lua script [13] and sorted sets data structure in Redis

3. Synchronization issue  
 Centralized data store like Redis

## Key components
1. Define the rate limiting policy
The first step is to determine the policy for rate limiting. This policy should include the maximum number of requests allowed per unit of time, the time window for measuring requests, and the actions to be taken when a limit is exceeded (e.g., return an error code or delay the request).
   1. Rules database
   1. Rules retriver service
   1. Rules cache
1. Store request counts: 
The rate limiter API should keep track of the number of requests made by each client. One way to do this is to use in memory cache like Redis. It is quick and supports the already implemented time-based expiration technique.
1. Client identifier builder:
This component generates a unique ID for a request received from a client. This could be a remote IP address, login ID, or a combination of several other attributes, due to which a sequencer can’t be used here. This ID is considered as a key to store the user data in the key-value database. So, this key is passed to the decision-maker for further service decisions.
## Algorithms
1. Token bucket algorithm
Pros:
This algorithm allows a burst of traffic as long as there are enough tokens in the bucket.
It is space efficient. The memory needed for the algorithm is nominal due to limited states.
Cons:
Choosing an optimal value for the essential parameters is a difficult task.
3. Leaking bucket algorithm
   1. When a request arrives, the system checks if the queue is full. If it is not full, the request is added to the queue.  
   1. Otherwise, the request is dropped.  
   1. Requests are pulled from the queue and processed at regular intervals.
  
   Leaking bucket algorithm takes the following two parameters:  
   Bucket size: it is equal to the queue size. The queue holds the requests to be processed at a fixed rate.
   Outflow rate: it defines how many requests can be processed at a fixed rate, usually in seconds.

    Pros:  
    1. Memory efficient given the limited queue size.
    1. Requests are processed at a fixed rate therefore it is suitable for use cases that a stable outflow rate is needed.
    
    Cons:      
    1. A burst of traffic fills up the queue with old requests, and if they are not processed in time, recent requests will be rate limited.
    1. There are two parameters in the algorithm. It might not be easy to tune them properly.

4. Fixed window counter algorithm
    1. The algorithm divides the timeline into fix-sized time windows and assign a counter for each window.
    1. Each request increments the counter by one.
    1. Once the counter reaches the pre-defined threshold, new requests are dropped until a new time window starts.
  
    Pros:  
    1.Memory efficient.   
    1.Easy to understand.    
    1.Resetting available quota at the end of a unit time window fits certain use cases.
    
    Cons:      
    1. Spike in traffic at the edges of a window could cause more requests than the allowed quota to go through.
6. Sliding window log algorithm
   1. The algorithm keeps track of request timestamps. Timestamp data is usually kept in cache, such as sorted sets of Redis [8].
   1. When a new request comes in, remove all the outdated timestamps. Outdated timestamps are defined as those older than the start of the current time window.   
   1. Add timestamp of the new request to the log.   
   1. If the log size is the same or lower than the allowed count, a request is accepted. Otherwise, it is rejected.

    Pros:    
    Rate limiting implemented by this algorithm is very accurate. In any rolling window, requests will not exceed the rate limit.  
    
    Cons:    
    The algorithm consumes a lot of memory because even if a request is rejected, its timestamp might still be stored in memory.

## Discuss extra
###Monitoring
After the rate limiter is put in place, it is important to gather analytics data to check whether the rate limiter is effective. Primarily, we want to make sure:

1. The rate limiting algorithm is effective.
1. The rate limiting rules are effective.

For example, if rate limiting rules are too strict, many valid requests are dropped. In this case, we want to relax the rules a little bit. In another example, we notice our rate limiter becomes ineffective when there is a sudden increase in traffic like flash sales. In this scenario, we may replace the algorithm to support burst traffic. Token bucket is a good fit here.
