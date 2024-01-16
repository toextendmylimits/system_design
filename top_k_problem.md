# Top K problem heavy hitter
Reference https://serhatgiydiren.com/system-design-interview-top-k-problem-heavy-hitters/#google_vignette

## Requirement

### Clarifying questions?
1. Is k a big number?
2. What's the scale of the system?
3. Can the time window change?
4. 
### Functional requirement
1. topK(k, startTime, endTime)

### Non-functional requirements
1. Available
2. Scalable
3. Low latency
4. Accurate(as accurate as possible)

## Initial Discussion

### Single host
We start with a single host. And let's assume the whole data set can be loaded into a memory of that single host. Use heap.
<img width="1148" alt="top_k_single_host" src="https://github.com/toextendmylimits/system_design/assets/10056698/f91786ff-6c8a-431d-b275-9414aa05a62c">

Not scalable. If events are coming with a high rate, single host will quickly become a bottleneck. So, we may want to start processing events in parallel.

### Multi host
How do we achieve this? One of the options is to introduce a load balancer.
<img width="1148" alt="top_k_hash_table_multi_host" src="https://github.com/toextendmylimits/system_design/assets/10056698/844c135f-b093-45f8-80a5-d1308b2bf36b">


Each event then goes to one of the hosts in a cluster. Let's call them Processor hosts. And because the same video identifier may appear on different Processor hosts, each
Processor needs to flush accumulated data to a single Storage host. It is a bit better now, we can process events in parallel.

### Data partitioning

Component Data partitioner
<img width="1141" alt="top_k_hash_table_multi_host_partition" src="https://github.com/toextendmylimits/system_design/assets/10056698/abbde89f-dcc9-46aa-8b87-9a28da635679">

Each host process a subset of data, and then generate a list of top k events. And the results are merged together.

By partitioning data, we increased both scalability and throughput.

Now if consider the time window say 1 minute: Processor hosts can accumulate data only for some period of time, let's say 1-minute, and will flush 1-minute data to the Storage host. So, the Storage host stores a list of heavy hitters for every minute. 

Great, but if we want to know heavy hitters for 1-hour or 1-day period, how can we build? 1-hour list from 60 1-minute lists?  The issue is we don't have all the data for the hour as data that wouldn't make into the top k list for each minute has been lost. 1-hour list from 60 1-minute lists? What should we do? Let's store all the data on disk and use batch processing framework(MapReduce) to calculate a top k

### Counter min sketch
<img width="1095" alt="top_k_count_min_sketch" src="https://github.com/toextendmylimits/system_design/assets/10056698/5e62f429-bca0-4cbb-912a-14ec3c11f42b">

probabilistic data structure that serves as a frequency table of events in a stream of data.

## High-level design
<img width="1191" alt="top_k_high_level_design" src="https://github.com/toextendmylimits/system_design/assets/10056698/1ed0c021-5887-4691-8cc0-67115554daae"> 

### API Gateway

We allocate a buffer in memory on the API Gateway host, read every log entry and build a frequency count hash table we discussed before. This buffer should have a limited size, and when buffer is full, data is flushed. If buffer is not full for a specified period of time, we can flush based on time.

### Distributed messaging system
Initially aggregated data is then sent to a distributed messaging system, like Apache Kafka. Internally Kafka splits messages across several partitions, where each partition can be placed on a separate machine in a cluster. At this point we do not have any preferences how messages are partitioned. Default random partitioning will help to distribute messages uniformly across partitions.

### Fast path
On the fast path, we will calculate a list of k heavy hitters approximately. And results will be available within seconds.

We have a service, let’s call it fast processor, that creates count-min sketch for some short time interval and aggregates data. And because memory is no longer a problem, we do not need to partition the data. Any message from Kafka can be processed by any Fast Processor host. Every time we keep data in memory, even for a short period of time, we need to think about data replication. Otherwise, we cannot claim high availability for a service, as data may be lost due to hardware failures. Because count-min sketch already produces approximate results, meaning we lose data already in some way, it may be ok if we lose data in some rare cases when host dies. As long as slow path does not lose data, we will have accurate results several hours later. Absence of replication greatly simplifies the system. 

Every several seconds Fast Processor flushes data to the Storage. Remember that count-min sketch has a predefined size, it does not grow over time. Nothing stops us from aggregating for longer than several seconds, we may wait for minutes. But this will delay final results. Another trade-off to discuss. 

The Storage component is a service in front of a database. And it stores the final top k list for some time interval, for example every 1 minute or 5 minutes. 
On the slow path, we will calculate a list of k heavy hitters precisely. And results will be available within minutes or hours, depending on the data volume.

### Slow path
On the slow path, we will calculate a list of k heavy hitters precisely. 

On the slow path we also need to aggregate data, but we want to count everything precisely.
One option is to let MapReduce do the trick. We dump all the data to the distributed file system, for example HDFS or object storage, for example S3. And run two MapReduce jobs, one job to calculate frequency counts and another job to calculate the actual top k list. Data is then stored in the Storage service.

Let’s Data Partitioner read batches of events and parse them into individual events. Partitioner will take each video identifier and send information to a correspondent partition in another Kafka cluster.

And we need a component that will read data from each partition and aggregate it further. Let’s call it a Partition Processor. It will aggregate data in memory over the course of several minutes, batch this information into files of the predefined size and send it to the distributed file system, where it will be further processed by MapReduce jobs. Partition Processor may also send aggregated information to the Storage service.

### User case walk through for slow path and fast path
1. Fast path
<img width="1094" alt="top_k_fast_path" src="https://github.com/toextendmylimits/system_design/assets/10056698/bacd18f0-c13d-4be6-940d-c2a60708d456">

2. Slow path
<img width="1067" alt="top_k_slow_path" src="https://github.com/toextendmylimits/system_design/assets/10056698/57fd71ef-40fa-405c-9158-fa6a89ff24ac">

We have Data Partitioner, a component that reads each message and sends information about each video to its own partition. Here, each blue cylinder represents a partition of the Distributed Messaging System. For example, the first message is scattered around partitions 1 and 2. The second message goes to partitions 2 and 3. The third message goes to the second partition. We have Partition Processors aggregate data for each partition. Each processor accumulates data in memory for let’s say 5 minutes and writes this data to files, which are stored in the Distributed File System. Frequency Count MapReduce job reads all such 5-minute files and creates a final list per some time interval, let’s say 1 hour. Top K MapReduce job then uses this information to calculate a list of k heavy hitters for that hour.

3. Map Reduce
<img width="1175" alt="top_k_map_reduce" src="https://github.com/toextendmylimits/system_design/assets/10056698/1981078a-5e71-40e3-8a3b-db467dd11e10">

4. Data Retrieveal
<img width="1094" alt="top_k_data_retrieval" src="https://github.com/toextendmylimits/system_design/assets/10056698/d1785ed0-787e-4679-aa50-428c048843e4"> 

