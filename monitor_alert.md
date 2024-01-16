# Minitor and alerting

## Requirement

### Clarifying questions
1. Who are we building the system for? Are we building an in-house system for a large corporation like Facebook or Google, or are we designing a SaaS service?
2. Which metrics do we want to collect?
3. What is the scale of the infrastructure we are monitoring with this system?
4. How long should we keep the data?
5. May we reduce the resolution of the metrics data for long-term storage?
6. What are the supported alert channels?

### Functional requirements
The infrastructure being monitored is large-scale.

100 million daily active users

Assume we have 1,000 server pools, 100 machines per pool, 100 metrics per machine => ~10 million metrics

1-year data retention

Data retention policy: raw form for 7 days, 1-minute resolution for 30 days, 1-hour resolution for 1 year.

A variety of metrics can be monitored, for example:

CPU usage

Request count

Memory usage

Message count in message queues

### Non-Functional requirements
1. Available
2. Scalable
3. Low Latency

## High-level design

### Main components
1. Data collection: collect metric data from different sources.
1. Data transmission: transfer data from sources to the metrics monitoring system.
1. Data storage: organize and store incoming data.
1. Alerting: analyze incoming data, detect anomalies, and generate alerts. The system must be able to send alerts to different communication channels.
1. Visualization: present data in graphs, charts, etc. Engineers are better at identifying patterns, trends, or problems when data is presented visually, so we need visualization functionality.

### Data model
A metric name, a list of tag/labels, a list of timestamp and values
CPU.load host=webserver01,region=us-west 1613707265 76

### Data storage system
Time series database

### Design diagram
<img width="1090" alt="byte_metric_high_level_design" src="https://github.com/toextendmylimits/system_design/assets/10056698/47c014f8-cf10-4c31-a076-2d4047195d46">

1. Metrics source. This can be application servers, SQL databases, message queues, etc.
1. Metrics collector. It gathers metrics data and writes data into the time-series database.
1. Time-series database. This stores metrics data as time series. It usually provides a custom query interface for analyzing and summarizing a large amount of time-series data. It maintains indexes on labels to facilitate the fast lookup of time-series data by labels.
1. Query service. The query service makes it easy to query and retrieve data from the time-series database. This should be a very thin wrapper if we choose a good time-series database. It could also be entirely replaced by the time-series database’s own query interface.
1. Alerting system. This sends alert notifications to various alerting destinations.
1. Visualization system. This shows metrics in the form of various graphs/charts.


## Detailed design/dive deep

### Push vs Pull
There are two ways metrics data can be collected, pull or push. 

1. Pull
A list of metric collectors pull data from different source.

    1. Zookeeper
    Use Apache Zoopkeeper for service delivery. Service discovery contains configuration rules about when and where to collect metrics 
    
    The metrics collector fetches configuration metadata of service endpoints from Service Discovery. Metadata include pulling interval, IP addresses, timeout and retry parameters, etc.
    
    The metrics collector pulls metrics data via a pre-defined HTTP endpoint (for example, /metrics). To expose the endpoint, a client library usually needs to be added to the service. In Figure 10, the service is Web Servers.
    
    Optionally, the metrics collector registers a change event notification with Service Discovery to receive an update whenever the service endpoints change. Alternatively, the metrics collector can poll for endpoint changes periodically.
    
    1. Consitent hash for assigning metrics collectors
    At our scale, a single metrics collector will not be able to handle thousands of servers. We must use a pool of metrics collectors to handle the demand.One common problem when there are multiple collectors is that multiple instances might try to pull data from the same resource and produce duplicate data. There must exist some coordination scheme among the instances to avoid this.
    
    One potential approacho designate each collector to a range in a consistent hash ring, and then map every single server being monitored by its unique name in the hash ring.

1. Push
In a push model, a collection agent is commonly installed on every server being monitored. A collection agent is a piece of long-running software that collects metrics from the services running on the server and pushes those metrics periodically to the metrics collector. The collection agent may also aggregate metrics (especially a simple counter) locally, before sending them to metric collectors.

<img width="1047" alt="byte_metric_push" src="https://github.com/toextendmylimits/system_design/assets/10056698/b7f2f595-02a0-4010-a89d-4d623c7a9993">

1. Pull or push?

- Easy debugging	
The /metrics endpoint on application servers used for pulling metrics can be used to view metrics at any time. You can even do this on your laptop. Pull wins.

If the metrics collector doesn’t receive metrics, the problem might be caused by network issues.

- Short-lived jobs		
Some of the batch jobs might be short-lived and don’t last long enough to be pulled. Push wins. This can be fixed by introducing push gateways for the pull model 
