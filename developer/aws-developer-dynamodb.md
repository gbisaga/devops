DynamoDB - NoSQL serverless database
- NoSQL - Non relational databases, distributed - scale horizontally - no joins, all data required is in one row
- Highly available, replication across 3 AZ - scales to massive workloads, millions of requests/sec
- Fast and consistent performance - low latency on retrieval
- Integrated with IAM for security, auth, admin
- Event driven programming with DynamoDB streams
- Made of tables with primary key - infinite number of items (=rows) - each item has attributes, can be added, can be nested
- Max size of an item is 400KB - pack a lot of data, this is encouraged
- Scalar types plus List, Map, Set as document types

Primary keys:
- Option 1: partition key (hash) only - must be unique per item, must be "diverse" (high cardinality) so data is distributed. E.g. user id is partition key
- Option 2: partition key + sort key - combination must be unique - partition key groups the data, then sorted by sort key. user is is partition, sort key is user's game id.
- Example: movie datbase - use movie_id, not language or producer

Provisioned throughput - IMPORTANT FOR EXAM
- Have to say your provisioned read and write capacity units
- Independently scale Read Capacity Units from Write Capacity Units
- Option to set up autoscaling
- Thruput has "burst credit" to exceed temporarily - if burst credit empty, get ProvisionedThroughputExceeded exception - use exponential backoff retry
- 1 WCU = 1 writesec up to 1KB in size - larger, more WCUs
  - 10 obj/sec, 2KB each = 20 WCU
  - 6 obj/sec, 4.5 KB each - 6x5 = 30 WCUs
  - 120 obj/min, 2 KB each - 2x2 = 4 WCUs
- Reads - either strongly consistent or eventually consistent reads 
  1 RCU = 1 strongly consistent read/sec or 2 eventually consistent read/s, for up to 4KB item size
  - 10 strongly consistent reads/sec of 4KB each = 10 RCU
  - 16 eventually reads/sec of 12KB = 8x3 = 24 RCU
  - 10 strongly consistent read/sec of 6KB = 10x2 = 20 RCU (round up 6KB)
- WCU and RCU are spread evenly among partitions
- if get ProvisionedThroughputExceeded -
  - hot keys (one partition key used a lot, e.g. sale item)
  - how partition - bad key
  - very large items - RCU/WCU depend on size of items
  - Solution: exponential backoff (in SDK), distribute partition keys, if RCU can use DynamoDB Accelerator (DAX - a cache)
- Autoscaling - nice especially to determine usage

DynamoDB APIs
- Writing data
  - PutItem - write, create or rull replace
  - UpdateItem - partial update of attributes - can also use Atomic counters and increase them
  - Conditional writes - accept write/update only if conditions are respected, otherwise reject - concurrent access, no performance impacted
- Deleting data
  - DeleteItem - delete a row, can be conditional
  - DeleteTable - much quicker and cheaper than calling DeleteItem on all items. Just delete and re-create the table.
- BatchWriteItem - up to 25 put and/or delete in one call
  - up to 16MB written
  - batching save in latency, and more efficient because done in parallel
  - possible for part of batch to fail - retry failed items only using exponential backoff
- Read data
  - GetItem - read on primary key (partition or partition+sort)
  - Eventually consistent by default - option to use strongly consistent (more RCU - take longer)
  - ProjectionExpression - only certain attributes - save network
  - BatchGetItem - up to 100 items, 16MB of data - retrieve in parallel. One or more items from one or more tables. You identify requested items by primary key.
- Querying data
  - Partition key must be =, sort key have expressions like >, <
  - FilterExpression to further filter - client side
  - up to 1MB data, number specified in Limit with paging
  - query a table, local secondary index, global secondary index
- Scan - inefficient way of querying - uses lots of RCU
  - reads entire table - returns up to 1MB data, page
  - consumes a lot of RCU - limit using Limit and pause
  - Parellel scans - faster, but lots of RCU
  - Can use ProjectionExpression plus FilterExpression

Second indexes
- Local secondary index = alternate sort key
  - Must be defined at table creation time
  - Up to 5 LSIs per table
- GSI another table, uses its own RCU/WCU = alternate hash key
  - Can add at any time

If writes are throttled on GSI, then main table will be throttled
- Choose GSI partition key carefully
- Assign WCU capacity carefully
- LSI has no special throttling

Concurrency models - optimistic locking

DynamoDB Accelerator - DAX
- Create a DAX cluster
- Seamless cache, microsecond latency
- Solves hot key problem
- 5 min TTL by default
- Up to 10 nodes/cluster, Multi AZ
- Second (KMS, VPC, IAM, CloudTrail)

DynamoDB streams
- all create, update, delete - changelog ends up in a stream
- read stream by Lambda -> send emails, analytics, create derivative tables, insert into other DBs, etc.
- cross region replication
- stream has 24 hr data retention (no config)

DynamoDB TTL
- define a TTL column with an expiry date 
- deletions do not use WCU/RCU - background task
- deletes with 48 hours of expiration; usually done pretty
- also deleted from indexes
- will create a DynamoDB stream event
- create a number column, set to epoch time in seconds since 1970
- set TTL attribute's name in Enable TTL dialog

DynamoDB CLI
- --projection-expression - attributes to retrieve
- --filter-expression - 
- general CLI pagination including S3
  - purely an optimization: --page-size - request less data, e.g. to avoid timeouts
  - --max-items - max number of items, returns NextToken send with --starting-token

DynamoDB Transactions:
- New "transactional" write and read mode
- Ability to CUD multiple rows in multiple tables at the same time, "all or nothing" - e.g. update bank transaction and account balance tables at once
- Consume 2x of WCU/RCU

DynamoDB security and other features
- VPC endpoints available
- IAM, KMS, SSL
- Backup and restore - point in time restore - no performance impact
- Global tables - fully replicated, high performance
- DMS to migrate other DBs to DynamoDB
- Launch local DynamoDB on your computer for development or testing
