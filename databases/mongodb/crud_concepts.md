# Transactions
## Transactions and atomicity
Atomicity: writing to a single document is always atomic. it is best to follow the denormalised schema while designing mongodb schemas to minimise usage 
of transactions so that the entire document is updated atomically

If we are trying to update/insert multiple docments, each insert will be atomic, but the overall insert/update will not.
For these use cases we need the transactions api (distributed transactions)
The distributed transactions api can be used for:
- update multiple docs in same collection
- across collections
- across databases
- across shards

Distributed transactions are atomic:
Either all changes or no changes are applied
Operations outside the txn will see the updated changed only when the txn commits
If the txn rolls back, no changes will ever be visible

## Scenarios
### Stale reads inside transaction
Read operations inside a transaction are not guaranteed to see writes performed by other committed transactions or non-transactional writes
- A transaction is in-progress.
- A write outside the transaction deletes a document.
- A read operation inside the transaction can read the now-deleted document since the operation uses a snapshot from before the write operation.

To prevent this,we can use the findOneAndUpdate method
This will acquire a write lock on the document
Now, if some other operation tries to update/delete this doc, mongo will throw a writeconflict error on the external operation
once the txn is committed, mongo releases the lock

### Reads outside transaction
During the commit for a transaction, outside read operations may try to read the same documents that will be modified by the transaction
Outside reads that use read concern "snapshot" or "linearizable" wait until all writes of a transaction are visible.
Outside reads using other read concerns do not wait until all writes of a transaction are visible, but instead read the before-transaction version of the documents.


Limitations:
By default, a transaction must have a runtime of less than one minute.
By default, transactions waits up to 5 milliseconds to acquire locks required by the operations in the transaction. If the transaction cannot acquire its required locks within the 5 milliseconds, the transaction aborts.
Transactions release all locks upon abort or commit.


### Distributed queries:
https://www.mongodb.com/docs/manual/core/distributed-queries/
- Read preference - read from primary/primaryPreferred/secondary/nearest in replica set
- Write preference - writes are always directed to the primary and records the write operation in the primary's oplog(operation log). This oplog is then replicated to the secondary replicas which then apply the operations in an async manner

### Read concerns(read isolation levels)
- Default is read_uncommitted
- Write operations are atomic with respect to a single document; i.e. if a write is updating multiple fields in the document, a read operation will never see the document with only some of the fields updated. However, although a client may not see a partially updated document, read uncommitted means that concurrent read operations may still see the updated document before the changes are made durable.

#### Read concern comparison with kafka
readconcern : majority -> similar to the high watermark concept in kafka consumer
Do we want to see the latest data in the replica or the data that is sucessfully replicated to the majority of the replicas in replica set?

writeconcern/write acknowledgment : majority -> similar to acks in kafka producer
w: majority -> acks=all  
{ w: "majority" } is the default write concern for most MongoDB deployments.
consider a replica set with 3 voting members, Primary-Secondary-Secondary (P-S-S). For this replica set, calculated majority is two, and the write must propagate to the primary and one secondary to acknowledge the write concern to the client.
  
w: 1 -> acks=1
Requests acknowledgment that the write operation has propagated to the standalone mongod or the primary in a replica set.   

w:0 -> acks=0
Requests no acknowledgment of the write operation. Has the lowest latency
  
https://www.mongodb.com/docs/manual/faq/concurrency/

  
What do i need to follow up in our mongo cluster?
  what is the readConcern?: majority
  what is readPreference? guess: primaryPreferred


## Final Notes
- All reads and writes in a transaction have to take an intent exclusive(IX) lock on the collections they are accessing
- If the collection is already locked by an S(shared) or X lock, then the current txn will wait for default of 5 ms before failing

- Write conflicts 
![image](https://github.com/soniamartis/system-design/assets/12456295/b51c59e5-26bc-4358-99d9-2d0c6845eedf)

- If a write conflict is thrown in a txn, then it has to be programatically retried, mongo will not retry automatically
- If a write conflict is thrown on a write operation that is not within txn, then it will be automatically retried till it succeeds
- To emulate the SELECT .. FOR UPDATE behaviour, we need to use findAndModify that will take a write lock on a document, so that it is not accessible to other operations and it will throw a write conflict in external operations

