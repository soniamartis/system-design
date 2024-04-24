# Change Data Capture

CDC is a set of technologies/ processes that identify the changes in the source DB and apply those changes to the target

## Usecases
Can be used to implement replication of datasets/ copying the data from one kind of DB to another
eg: read data from sybase and write it to datalake(HDFS files)
Instead of running batch scripts and scheduling them to run at certain intervals of time(can be very long running based on source), we can use CDC to detect changes in the source DB and 
apply those incremental changes to targets(1 or more)

## Different ways to implement CDC
- Database triggers: this can prove useful for smaller systems where we want to maintain audit for a particular tables
- Change data streams option provided by vendor DB like change data streams in mongodb: https://www.mongodb.com/developer/languages/java/change-streams-in-java/

  ![image](https://github.com/soniamartis/system-design/assets/12456295/c26eaa39-d91d-496e-934c-c9af5cd74bb1)
- Use open-source technologies like Debezium that have connectors

### NOTE:
Debezium mongodb connector uses mongodb change streams under the hood to read the oplog data from the primary in the replicaSet


Ideas:
For the validation framework, if we want to maintain cache and not hit DB everytime for getting the validations while running batches, we can create cache, and refresh it once a day
whenever a user makes an update to the validation collection, we could use mongodb change stream mechanism to pick the change and accordingly refresh cache
POC required to check whether the changes will be picked by all listeners or only one. should be all tbh!!



  

