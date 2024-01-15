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

## Design

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

Each host process a subset of data, and then generate a list of top k events. And the results are merged together.
