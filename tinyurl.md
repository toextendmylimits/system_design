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
