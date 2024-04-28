HDD performance characteristics

HDD perf is measured using 2 groups: access time and data transfer time
- The platter is made up of 2 things, tracks and sectors
- Every track is divided into sectors

----

### Access time is broken into
- Seek time (head moves to correct track)
- Rotational latency (disk spins to correct sector on the track)


![image](https://github.com/soniamartis/system-design/assets/12456295/952383cd-292e-41d7-a4b4-9099cf80fe6c)

### Seek time:
Time taken for the head on the actuator arm to move to the correct track on the platter where data is located
- If initial track is the outermost track and the data is located on the innermost track -> seek time will be maximum
- If initial track and desired track are the same -> seek time is 0

### Rotational latency
Time taken for the disk to spin in order to get the correct sector under the head. It is measured in rpm(revolutions per minute)
  

----
## Data Transfer time aka throughput

Time taken to transfer data from the disk surface to the host system
Consists of internal rate(time taken to move data from disk surface to conrtoller on the drive) and external rate (time taken to move data from controller on drive to host system)

### Why is sequential I/O faster than Random I/O
- In seq IO, we just have to seek to the correct track on platter once and disk will rotate to get the correct sector under the disk head. After that, we keep reading sequentially from there onwards
- In case of random I/O, we have to seek several times to different tracks on disk and then read from the sectors on those tracks
- Avg seek time for an HDD is 10 ms
- In short, sequential I/O is faster as we minimise the number of seeks
