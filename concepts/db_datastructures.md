# Data structures that power databases

| Data structure  | Database |
| ------------- | ------------- |
| Skip list  |  Redis - sorted set  |
| Hash indexes  | Redis, and internally in all kinds of dbs  |
| LSM Tree(memtable + SSTable + bloom filter) | Cassandra DB, RocksDB, LevelDB|
| B-tree/ B+ tree | MySql, Postgres, MongoDB|
| Inverted index | Elastic search|
| Suffix tree | Text-based search(TBD)|
| R-tree | MongoDB, elastic search for spatial queries|



## Hash Index

### Concepts
- data structure
- merging and compaction
- read/search pattern
- write pattern
- additional handling
- use-case
- limitations

### Data structure
- Works similar to a hash map for data that is stored in key-value format
- In-memory: hash map, on-disk: data
- The key is value of the key and the value is the location of the key-value pair on disk
- This index is held in memory and actual data is written to log segment on disk
- Every log segment maintains its own in-memory hash index
- There is also a on-disk backup for this in-memory index, in case of node failures
- Since we are using an append-only log for storing data, we can run out of disk space eventually

![image](https://github.com/soniamartis/system-design/assets/12456295/6d54e25d-53af-4d78-aab7-cc2fe2775d9e)


### Merging and compaction
- The log is divided into segments and data is written to these segments
- Since we keep appending to the log's segments, we will run out of disk space eventually
- To avoid this, we use compaction + merging technique
- The same key can be appended several times to the same segment, across segments
- Compaction: for given segment, keep only the latest value per key and throw away duplicate keys
- Merging: Now that we have several smaller segments, we merge them into larger segments
- During the process, the hash index for that segment is updated as well

![image](https://github.com/soniamartis/system-design/assets/12456295/e3697103-dd14-4b8f-92db-7b398dffc6bb)


### Read pattern
- Since we have multiple segments and a hash index per segment, we first start looking into the hash index of the latest segment, if key not found, look into the second most recent and so on
- Since the segments are compacted and merged, hopefully, there wont be too many segments to look into
- This emphasises the fact that this DS works best for smaller number of keys

### Write pattern
- Just append to the end of the active segment
- Update the hash index as well as the on-disk backup for this index

### Additional handling
- Record deletion : append a tombstone record for the key that u want to purge. In merge process, the tombstone record provides an indication that all previous value of this key can be discarded
- Crash recovery: If the db is restarted, all the in-memory hash indexes are lost. They can be reconstructed by going over every segment, but this can be time-consuming, and to avoid, we store a snapshot of each segment's hashmap on disk
- Partially written records: DB can crash halfway through a write. We can maintain a checksum to check for integrity of writes
- Concurrency control: Only one thread writes to the segment, and multiple threads can read from the segments, as these segments are read-only(only the last segment ie active segment is the one where data is actively appended)

### Use-case
- Best suited for workloads having smaller number of keys whose values are frequently updated, instead of large number of keys

### Limitations
- Range queries are not suitable, eg get all values in range 1-100000. We will have to lookup for every key in the raange 1-100000
- hash table must fit in memory for optimal performance, using an additional on-disk hash table will slow down queries as it will result in lot of random IO

  
  



