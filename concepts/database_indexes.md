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


## SSTables(Sorted string table) and LSM Trees (log-structured merge trees)

### Concepts
- data structure
- merging and compaction
- read/search pattern
- write pattern
- additional handling
- use-case
- limitations

### Data structure
- Uses a balanced tree like AVL/Red-black in memory and SSTable that is a sorted log on disk
- Writes are directed to this in-memory balanced tree, once it reaches a certain size, it is then flushed to the disk in sorted order as an SSTable
- In balanced tree, we can insert in any order and can read in sorted order
- Bloomfilter is used in addition mentioned below
- In SSTable, we can maintain a sparse index for efficient search instead of a hash index, since the data on sstable is sorted

### Merging and compaction
- SSTable segments are merged and compacted similar to the merge stage in a mergesort algorithm
- If 2 segments are being merged and if key appears in both segments, then the key from the more recent segment will be picked
- The same key does not repeat twice within the same segment

### Read/search pattern
- First seach for the key in memtable
- If not found, then search in the most recent SSTable segment and keep doing so till u find it
- This shows that search can slow down for keys that do not exist
- In this scenario, we can use bloom filter which is a data structure that gives a definite no, if they key does not exist and a prababilistic yes, if it does

### Write pattern
- When a write comes in, write it to an in-memory balanced tree data structure.This structure is called a memtable
- When memtable gets bigger than a threshold, say a few megabytes, write it out to disk as SSTable file
- The new sstable becomes the most recent segment of the database. While the sstable is being written out to disk, writes can continue to a new memtable instance

### Additional handling
- Server crash: The most recent writes ie. data in memtable might be lost. We can handle this by using an additional log to which we write the memtable data, and it doesnt need to be sorted, we use it just to restore the memtable after a crash, everytime the memtable is flushed on disk, the log can be discarded

### Use-case
- Can perform efficient range queries as data is sorted
- Has a high write throughput as data is written to disk sequentially

### Limitations
- Compaction can impact performance of ongoing reads/writes due to limited disk b/w - it can easily happen that a request needs to wait while disk finishes an expensive compaction operation
- compaction may not be able to keep up with the # of incoming writes, resultng in large # of unmerged segments, slowing down the reads

## B-Trees
- They keep key-value pairs sorted by key
- The DB is broken down into fixed size pages of 4KB each
- on-disk datastructure where root has maintains keys and reference to the next set of pages
- all intermediate nodes too maintain key and reference to next page
- leaf pages either contain the values for the keys or references pages where value will be found

![image](https://github.com/soniamartis/system-design/assets/12456295/59a59bdd-48fe-4abe-8a99-cd467dc02b59)


### Read pattern
- Keep traversing the references till u get to the page that has the data for that key
- eg. to find data for key 251, we first start at root, 251 will lie between 200-300, then we go to the page that has keys between 200-300, then we search in that page, now 251 will lie betweeen 250 and 270, so we go to the reference of 250 till we get to the last page that has 251
- Number of references to the child pages in one page of b-tree is called branching factor. in above example, it is 6. In reality, it can go upto several hundreds

### Write pattern
- To add a new key: search the btree to get to the page that encompasses the range for the key and write to that page. If the page is full, break it down to 2 half-full pages and update the parent to point to the 2 pages
- Update value of key: search for the laf page containing the key, change value in the page, and write page back to disk

![image](https://github.com/soniamartis/system-design/assets/12456295/e35358a4-c181-4cb9-a3d0-62ac82b1bac2)


### Additional handling
- a btree updates the pages on disk directly and does not use the append-only methodology of LSM trees, so is more susceptible to disk crashes
- a WAL(write-ahead log) is used where every modification to the b-tree is written, then the opertaion is carried out on b-tree pages. If there is a crash, this wal is used to restore the tree
- Concurrency control: implement light-weight locks on btree so that threads do not see an inconsistent state of the tree
  
 ### B-treee optimisations
 - Btree searches can slow down if there are many levels in the index, resulting in more random accesss. In order to decrease the number of levels, we need to increase the # of keys in each level(branching factor). This can be done by abbreviating the keys and packing in more keys within the pages
 - for range queries, if leaf pages are sequential, reads can be faster, but there is no such requirement that pages with nearby ranges will be nearby on disk. Attempts are made by db, to position these pages sequentially
 - Additional pointers to the tree: sibling pages have lefta dn right pointers so that we do not have to go back to the parent to get to the next/prev sibling page

| B-tree  | LSM Tree |
| ------------- | ------------- |
| faster for reads| faster for writes - reads can be slower due to searching in various data structures for the key|
| results in disk fragmentation and has higher write amplification| supports better compression and has lower write amplification, suited for ssds|
| - | compaction can impact performance of ongoing reads and writes, due to limited disk resources|
| key-value exists in exactly 1 place giving better txn isolation using locks on range of keys| - |



