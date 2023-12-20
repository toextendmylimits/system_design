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
