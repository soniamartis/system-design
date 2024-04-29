# Revise these numbers for back of the envelope estimations
## Approximations
- Number of seconds in a day = 10^5
- Avg disk seek time = 10 ms
- Read 1 MB sequentially from memory = 250 ps
- Read 1 MB sequentially from network = 10 ms
- Read 1 MB sequentially from disk = 30 ms

## Latency comparisons
CPU cache reference < memory reference < compression with zippy < read 1 MB seq from memomory < round trip within same DC < disk seek < read 1 MB sequentially from network < read 1 MB sequentially from disk < round trip between DCs

## Conclusions:
- Memory is fast but disk is slow
- Round trip in same DC is faster than disk access
- Compress data before sending over internet
- Avoid disk seeks if possible
- Simple compression algos are fast

## Storage calculations(using java as reference)
| data type  |  size |
| ------------- | ------------- |
| boolean  | 1 bit  |
| byte  | 1 byte  |
| char | 2 bytes|
| short | 2 bytes|
| int | 4 bytes|
| float | 4 bytes|
| long | 8 bytes |
| double | 8 bytes |
 
- Avg size of single document/record in database ~1 KB
- Avg size of media file ~1 MB


## Numbers to calculate
QPS(queries per second)
TPS(transactions per second)
storage requirements


