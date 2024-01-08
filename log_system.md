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

## Key components
1. Storage  
The logs need to be stored somewhere after accumulation. We’ll choose blob storage to save our logs.
1. Log accumulator:  
An agent that collects logs from each node and dumps them into storage. So, if we want to know about a particular event, we don’t need to visit each node, and we can fetch them from our storage.
1. Log indexer:  
The growing number of log files affects the searching ability. The log indexer will use the distributed search to search efficiently.
1. Visualizer:  
The visualizer is used to provide a unified view of all the logs.

## Initial design
<img width="1005" alt="logging_system" src="https://github.com/toextendmylimits/system_design/assets/10056698/adde2f9d-0d72-4afe-bf9c-8181241a3a39">

There are millions of servers in a distributed system, and using a single log accumulator severely affects scalability. Let’s learn how we’ll scale our system. 

## Improved design using pub-sub system
Each service will push its data to the log accumulator service. It is responsible for these actions:  
1. Receiving the logs.
1. Storing the logs locally.
1. Pushing the logs to a pub-sub system.  
We use the pub-sub system to cater to our scalability issue. Now, each server has its log accumulator (or multiple accumulators) push the data to pub-sub. The pub-sub system is capable of managing a huge amount of logs.

To fulfill another requirement of low latency, we don’t want the logging to affect the performance of other processes, so we send the logs asynchronously via a low-priority thread. 

The data does not reside in pub-sub forever and gets deleted after a few days before being stored in archival storage. However, we can utilize the data while it is available in the pub-sub system.  

<img width="1001" alt="logging_system_pub_sub" src="https://github.com/toextendmylimits/system_design/assets/10056698/8c125888-41d8-4858-91dc-9d8ac4be699f">

New key components identified:
1.Filterer:   
It identifies the application and stores the logs in the blob storage reserved for that application since we do not want to mix logs of two different applications.
1.Error aggregator:   
It is critical to identify an error as quickly as possible. We use a service that picks up the error messages from the pub-sub system and informs the respective client. It saves us the trouble of searching the logs.
1. Alert aggregator:  
Alerts are also crucial. So, it is important to be aware of them early. This service identifies the alerts and notifies the appropriate stakeholders if a fatal error is encountered, or sends a message to a monitoring tool.

In our design, we have identified another component called the expiration checker. It is responsible for these tasks:
Verifying the logs that have to be deleted. Verifying the logs to store in cold storage.
