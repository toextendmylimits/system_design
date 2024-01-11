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

3. Search Tweet  
searchTweet(user_id, search_term, max_result, exclude, media_field, expansions, sort_order, next_token, user_location)
Just explain user_id, search_term, max_result and user_location.

## Detailed design 
