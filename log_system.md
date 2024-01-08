# Design a distributed logging system
## Requirments
### Clarifying quesions
1. What is the key functionaltiy? To write logs from multiple microservice, and then be able to search the log?
2. How long shall we keep the logs?

### Funcitional requirement
1. Write logs  
Different micro services can write logs into the logging system
1. Store logs  
The logs should reside in a distributed storage for easy access
1. Search logs  
It should be simple to search logs.
1. Centralized logging visualizer  
The system should provide a unified view of globally separated services.

### Non-functional requirement
1. Low latency    
Logging is an I/O-intensive operation that is often much slower than CPU operations. We need to design the system so that logging is not on an application’s critical path.
1. Scalability:  
We want our logging system to be scalable. It should be able to handle the increasing amounts of logs over time and a growing number of concurrent users.
1. Availability:  
The logging system should be highly available to log the data.

# Design
## API
1. Write a message
write(unique_ID, message_to_be_logged) where unique id contains application id, service id and timestamp
2. Search
search(keyword)

## Key components and high-level design
1. Storage  
The logs need to be stored somewhere after accumulation. We’ll choose blob storage to save our logs.
1. Log accumulator:
An agent that collects logs from each node and dumps them into storage. So, if we want to know about a particular event, we don’t need to visit each node, and we can fetch them from our storage.
1. Log indexer:  
The growing number of log files affects the searching ability. The log indexer will use the distributed search to search efficiently.
1. Visualizer:  
The visualizer is used to provide a unified view of all the logs.
