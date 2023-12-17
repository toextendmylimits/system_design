# Design a distributed message queue
## Functional requirements:
1. Create Queue
2. Send message
3. Receive message
4. Delete queue

## Non-functional requirements:
1. Durability
   The data received by the system should be durable and shouldn’t be lost.
1. Scalibility
   The system needs to be scalable and capable of handling the increased load, queues, producers, consumers, and the number of messages.
3. Availability
   The system should be highly available for receiving and sending messages. It should continue operating uninterrupted, even after one or more of its components fail.
4. Performance
   High throughput and low latency
