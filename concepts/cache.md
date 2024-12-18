# Caching

## Caching strategies
![image](https://github.com/soniamartis/system-design/assets/12456295/2434f096-ae8f-48c8-87a5-a892f4aca07a)

We can combine strategies together too:
eg: 
- Cache aside(read) + write around(write)
- Read through(read) + write around(write)
- Read thru(read)+ write thru(write) - in this scenario we will not even need cache invalidation, as every write to cache will be written to DB as well eg DynamoDB Accelerator(DAX)

Difference between cache-aside and read-thru is that in the former approach, app is responsible for handling cache-miss
whereas in the latter, the cache itself is responsible for handling cache-miss

## Things to consider when using cache
- Types of cache: in-memory, distributed(cache distributed across nodes) and client-side caching
- Expiration time(TTL): time when the cache entries should be expired and refreshed so that we do not serve stale data
- Cache effectiveness: hit rate vs miss rate, eviction rate (high eviction rate means cache is too small or eviction policy is not too suited)
- Data access patterns: is app read-heavy vs write-heavy in order to chose the right caching strategy

## Articles
https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/

