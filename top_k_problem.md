# Top K problem heavy hitter

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

### High-level design
<img width="1191" alt="top_k_high_level_design" src="https://github.com/toextendmylimits/system_design/assets/10056698/1ed0c021-5887-4691-8cc0-67115554daae"> 


