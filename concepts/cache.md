# Caching

## Caching strategies
![image](https://github.com/soniamartis/system-design/assets/12456295/2434f096-ae8f-48c8-87a5-a892f4aca07a)

We can combine strategies together too:
eg: 
- Cache aside(read) + write around(write)
- Read through(read) + write around(write)

Difference between cache-aside and read-thru is that in the former approach, app is responsible for handling cache-miss
whereas in the latter, the cache itself is responsible for handling cache-miss

