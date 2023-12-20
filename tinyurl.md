# Url shortening service
## Requirement
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

   301 redirect - permanent redirect  
   302 redirect - temporary redirect

3. Links will expire after a standard default time span.

### Non-functional requirement
1. The system should be highly available. This is really important to consider because if the service goes down, all the URL redirection will start failing.
1. Scalability: Our system should be horizontally scalable with increasing demand.
1. URL redirection should happen in real-time with minimal latency.
1. Shortened links should not be predictable.
