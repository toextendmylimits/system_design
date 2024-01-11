# Design twitter
## Clarifying questions
1. What are the main functionlaties? To post tweets, reply tweets? search twitter?
2. What's the scale of the system such how many daily active users we need to support?

## Functional requirement
1. User registration/login/authentication
2. Post Tweets: Registered users can post one or more Tweets on Twitter.
3. Reply to Tweets: Registered users can reply to the public Tweets on Twitter.
4. Follow & unfollow people
5. View timeline

Secondary
7. Search Tweets: Registered users can search Tweets by typing keywords, hashtags, or usernames in the search bar on Twitter.
8. Delete Tweets: Registered users can delete one or more of their Tweets on Twitter.
9. Like or dislike Tweets: Registered users can like and dislike public and their own Tweets on Twitter.
10. Retweet a Tweet: Registered users can Retweet public Tweets of other users on Twitter.

## Non-functional requirement
1. Highly available
2. Low latency
3. Scalability
4. Reliability

## High-level design
<img width="714" alt="twitter_high_level" src="https://github.com/toextendmylimits/system_design/assets/10056698/753f649e-f4de-456e-9824-902c4f20e7f0">

### APIs
1. Post Twitter   
postTweet(user_id, access_type, tweet_type, content, tweet_length, media_field, post_time, tweet_location, list_of_used_hashtags, list_of_tagged_people)
user_id
<img width="840" alt="api_post_twitter" src="https://github.com/toextendmylimits/system_design/assets/10056698/fa50c42e-61ad-4a11-be09-57dd3f7ff0a1">

2. Reply to Tweet  
The /replyTweet API is used when users reply to public Tweets.

replyTweet(user_id, tweet_id, tweeted_user_id, reply_type, reply_content, reply_length)
The reply_type, reply_content, and reply_length parameters are the same as tweet_type, content, and tweet_length respectively.

3. view timeline
viewTimeline

## Detailed design 
### Storage
1. Twitter accounts, twitter messages
Canssandra
2. Blob store
For images and videos that attache to a tweet
3. Graph database for followers/followees
Twitter stores this relationship in the form of a graph. Twitter used FlockDB, a graph database tuned for huge adjacency lists, rapid reads and writes, and so on, along with graph-traversal operations

<img width="1002" alt="twitter_detailed_design" src="https://github.com/toextendmylimits/system_design/assets/10056698/cd132956-e8b0-46c3-8dae-947b3136f6ec">

### Key components
1. Tweet service:   
When end users perform any operation, such as posting a Tweet or liking other Tweets, the load balancers forward these requests to the server handling the Tweet service. Consider an example where users post Tweets on Twitter using the /postTweet API. The server (Tweet service) receives the requests and performs multiple operations. It identifies the attachments (image, video) in the Tweet and stores them in the Blobstore. Text in the Tweets, user information, and all metadata are stored in the different databases (Manhattan, MySQL, PostgreSQL, Vertica). Meanwhile, real-time processing, such as pulling Tweets, user interactions data, and many other metrics from the real-time streams and client logs, is achieved in the Apache Kafka.

Later, the data is moved to the cloud pub-sub through an event processor. Next, data is transferred for deduping and aggregation to the BigQuery through Cloud Dataflow. Finally, data is stored in the Google Cloud Bigtable, which is fully managed, easily scalable, and sorted keys.

1. Timeline service:
Assume the user sends a home timeline request using the /viewHome_timeline API. In this case, the request is forwarded to the nearest CDN containing static data. If the requested data is not found, it’s sent to the server providing timeline services. This service fetches data from different databases or stores and returns the Top-k Tweets. This service collects various interactions counts of Tweets from different sharded counters to decide the Top-k Tweets. In a similar way, we will obtain the Top-k trends attached in the response to the timeline request.

1. Search service:  
When users type any keyword(s) in the search bar on Twitter, the search request is forwarded to the respective server using the /searchTweet API. It first looks into the RAM in Apache Lucene to get real-time Tweets (Tweets that have been published recently). Then, this server looks up in the index server and finds all Tweets that contain the requested keyword(s). Next, it considers multiple factors, such as time, or location, to rank the discovered Tweets. In the end, it returns the top Tweets.
