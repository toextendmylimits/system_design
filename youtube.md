# Design youtube
## Clarifying questions
1. What are the main features we need to support?
2. What clients do we need to support?
3. How many daily active users?
4. What is the average daily time spent on the product?
5. Do we need to support different video resolutions?
## Requirement
1. Functional requirements
    1. Stream videos
    1. Upload videos
    1. Search videos according to titles
    1. Like and dislike videos
    1. Add comments to videos
    1. View thumbnails
1. Non-functional requirements  
    It’s important that our system also meets the following requirements:
    
    1. High availability: The system should be highly available. High availability requires a good percentage of uptime. Generally, an uptime of 99% and above is considered good.
    1. Scalability: As the number of users grows, these issues should not become bottlenecks: storage for uploading content, the bandwidth required for simultaneous viewing, and the number of concurrent user requests should not overwhelm our application/web server.
    1. Good performance: A smooth streaming experience leads to better performance overall.
    1. Reliability: Content uploaded to the system should not be lost or damaged.

## Resource estimation
1. Storage  
Let's assume the daily active users are 5 million, 10% of users upload video one day, each user watchs 5 videos, each video is 300MB, then total daily storage is 5 million * 10 % * 300M = 150 TB

