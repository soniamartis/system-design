# Load Balancer Algorithms

## Load Balancer
Used to route client requests to set of servers, belonging to the same service

## Algorithms
- Round Robin
- Weighted Round Robin
- Resource based
- Least connections
- Least time
- IP/URL hash


 ### Round Robin
 - Client requests are routed to servers in rotation
 - eg: if we have 3 app servers and 4 requests, requests will be routed to each server A, B , C, A
 - Best suited when the servers have equal processing abilities in terms of CPU/ memory and network B/W

### Weighted Round Robin
- Weights are assigned to the servers based on their processing capabilities and the requests are routed by weight
- RR is a special case of Weighted RR, where each server has the same weight
- eg Suppose we have 3 app servers A, B, C with weights 0.5, 0.25, 0.25, then 50% requests will be routed to A, and 25% each to B and C


### Resource based (Adaptive)
- Here the LB itself determines the weights that need to be assigned to servers based on certain health checks/ status indicator criteria
- Agents are installed on the server for extracting the status of server based on which the LB will decide the weights and route requests accordingly


### Least connections (Dynamic)
- Client request is routed to the server that has the least number of active connections
- This is suitable where all servers hve similar processing capabilities

### Least time
- Request is routed to the server that serves requests faster
- This is suitable for scenarios where the response time is of utmost importance

### IP/ URL Hash
- Hash the source IP address and allocate the client to a server
- Used when successive requests from a client need to be sent to same server
- In url hash, a hash key in generated on the url, so that the same back-end server serves request for a given url

  





