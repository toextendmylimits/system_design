# Url shortening service
Refer https://www.geeksforgeeks.org/system-design-url-shortening-service/

## Requirement
## Clarifying questions?
1. What is the key requirement? Is it to create alias of a long url with shorter length, and when clicking on the short url, it will be redirected to the original long url?
2. Hong long is the short url?
3. What characters are allowed in the shortened url? Is it safe to assume combinations of numbers 0 - 9 and lowercase and upperase alphabetic letters, i.e. a-z A-Z?
4. What is the traffic volume, such as how many short URLs to generate in a day?

## Resource Estimation
If we assume 100million URLs are generated a day, aaverage URL length is 1000 characters, and we store URL for 10 years.
1. Write operation per second is 100 million / 86400 seconds = 1160 per second. 
2. Assume ratio of read operation to write operation is 10, then read operation per second is 11600
3. Then total storage for 10 years is: 10years * 365day * 100million * 1000Bytes = 365TB
### Functional requirement
1. Url shortening
    POST api/v1/data/shorten
   
    request parameter: {longUrl: longURLString}
    
    return shortURL
    
    URL redirecting. To redirect a short URL to the corresponding long URL, a client sends a GET request. The API looks like this:
    
    GET api/v1/shortUrl
    
    Return longURL for HTTP redirection
 
2. Url redirecting  
   Figure 1 shows what happens when you enter a tinyurl onto the browser. Once the server receives a tinyurl request, it changes the short URL to the long URL with 301 redirect.

   1. 301 redirect - permanent redirect  
   A 301 redirect shows that the requested URL is “permanently” moved to the long URL. Since it is permanently redirected, the browser caches the response, and subsequent requests for the same URL will not be sent to the URL shortening service. Instead, requests are redirected to the long URL server directly.

   1. 302 redirect - temporary redirect
    A 302 redirect means that the URL is “temporarily” moved to the long URL, meaning that subsequent requests for the same URL will be sent to the URL shortening service first. Then, they are redirected to the long URL server.

    Each redirection method has its pros and cons. If the priority is to reduce the server load, using 301 redirect makes sense as only the first request of the same URL is sent to URL shortening servers. However, if analytics is important, 302 redirect is a better choice as it can track click rate and source of the click more easily.

3. Links will expire after a standard default time span.

### Non-functional requirement
1. The system should be highly available. This is really important to consider because if the service goes down, all the URL redirection will start failing.
1. Scalability: Our system should be horizontally scalable with increasing demand.
1. URL redirection should happen in real-time with minimal latency.
1. Shortened links should not be predictable.

### Capacity estimation
Storage/Query Rate/Bandwith

Let’s assume our service has 30M new URL shortenings per month. Let’s assume we store every URL shortening request (and associated shortened link) for 5 years. For this period the service will generate about 1.8 B records. 30 million * 5 years * 12 months = 1.8 B

Consider the average long URL size of 2KB ie for 2048 characters.
Short URL size: 17 Bytes for 17 characters  
created_at- 7 bytes  
expiration_length_in_minutes -7 bytes  

Total storage per month 30 million * 2kB = 60 GB, so each year, 0.7 TB, and 5 years 3.5 GB.

## APIs
1. Shortening a URL  
   shortURL(api_dev_key, original_url, custom_alias=None, expiry_date=None)
3. Redirecting a short URL
   redirectURL(api_dev_key, url_key)
1. Deleting a short URL
   deleteURL(api_dev_key, url_key)

