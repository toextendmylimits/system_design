# Url shortening service
## Requirement
### Functional requirement
1. Url shortening
 POST api/v1/data/shorten

request parameter: {longUrl: longURLString}

return shortURL

2.URL redirecting. To redirect a short URL to the corresponding long URL, a client sends a GET request. The API looks like this:

GET api/v1/shortUrl

Return longURL for HTTP redirection
3. Url redirecting