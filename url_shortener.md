# URL shortener

## Concepts
- sequencer generates uniqueIds
- encoder is used to make uniqueIds human-readable
- output of an encoder is always unique for a given input and the encoded data can then be decoded to get back the input value, which is not true in case of hashing
- hashing is only 1 way ie, we cannot get back the input from the hash value, whereas encoding/decoding is bi-directional, ie we can get back the input by decoding
- to convert a number from base x to base 10, always multiply by x^n eg: (abc)58 => a*58^2 + b*58^1 + c*58^0 = 113019
- to convert a number from base 10 to base x, keep dividing by x and then use remainders for generating the number
![image](https://github.com/soniamartis/system-design/assets/12456295/034e2783-d8da-4392-aba5-4daac9c65901)
![image](https://github.com/soniamartis/system-design/assets/12456295/ab686ee4-1fd1-466f-8bf8-13578eca2272)
- bloom filters can be used to check whether a given long url exists or not
- 301 redirect means a resource has permanently moved to a new location whereas 302 means it has moved temporarily. If we return a 301, the browser caches this redirect url and will no longer hit the url generation service and will directly redirect to the longUrl, if we return 302, this info is not cached, and browser will hit the url service everytime the shortUrl is provided


## Gather requirements
- 100 million urls generated per day
- characters in shortUrl = [0-9a-zA-Z] and as short as possible
- urls can be deleted after 5 years
- service will run for 10 years

## Functional requirements
- Given a long url, generate a short url
- Given a short url, return the long url
- Delete the mappings older than 5 years(purely backend functionality not exposed to users)

## Non functional requirements
- High availability, scalability, fault tolerance

## Back of the envelope estimations
- Write QPS:
- Number of urls generated per day = 100 million = 10^8
- Number of urls generated per second = 10^8/10^5 = 10^3 = 1k

- Read QPS
- Assume read:write ratio is 10:1
- Number of redirects = 10k

- Storage estimations
- #of urls generated in 10 years = 100 million entries * 365 days * 10 years = 365 billion records
- 1 entry ~ 1KB , therefore storage size = 365 * 10^9 * 10^3 = 365 * 10^12 = 365 TB

## System APIs
- POST api/v1/shorten?longUrl=xyz
- GET api/v1/{shortId}

eg:
- POST https://www.tinyurl.com/shorten?longUrl=https://en.wikipedia.org/wiki/Systems_design
- GET https://www.tinyurl.com/abcdEfg

## Data models and DB
mapping of longUrl and shortUrl
since we need to delete mappings older than 5 years, add timestamp to track when an antry was added
primary key of table

NoSql DB would be good fit like MongoDB as it can support large number of writes, it also supports a TTL index that can be created for auto-deletion of entries after 5 years
Data model:
id: UUID -> primaryKey
longUrl: String -> unique index
shortUrl: String -> unique index
creationTime: Instant -> ttl index

###  Logic:
- Given a long Url check wheher a shortUrl already exists for the same
- If yes, return shortUrl
- If no, generate shortUrl using urlShortener service, store mapping in collection and return

### Steps to generate shortened url:
- shortUrl = encode(uniqueId(longUrl))
- uniqueId(longUrl) does not really depend on contents of the longUrl, rather





