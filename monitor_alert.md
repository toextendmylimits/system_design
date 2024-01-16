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
