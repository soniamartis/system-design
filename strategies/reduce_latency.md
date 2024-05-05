## Strategies to reduce latency


Latency can be present at various levels like database, processing and network
- Database indexing
- Caching
- Load balancing
- Async processing
- CDN(s)
- Data compression

## Database

### Database indexing
- Make sure to create the right indexes
- Used to optimise slow running queries

## Processing  

### Caching
- Slow frequenctly accessed data in cache
- Speed up slow DB lookups

 ### Async processing
 - Do not block the request thread for long running tasks, rather carry it out in the background - executor service/ pub-sub mechanisms

## Network

### Load balancing
- Distribute load evenly aceoss servers
- Use the right LB type and algorithm

### Content delivery network
- Cache static content close to end users
- Reduce geographical distance to reduce latency

### Data compression
- Compress data before sending it over the network
