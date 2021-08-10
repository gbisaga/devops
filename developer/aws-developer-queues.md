----
SQS - queues
SNS - pub/sub
Kinesis - streaming, big data

Summary: https://campfirecode.medium.com/sqs-vs-sns-vs-kinesis-which-aws-messaging-service-to-use-1fa3aa6be97d

SQS - scales from 1 to 10k messages/second
- Default retention = 4 days, max = 14 days
- No limit to how many messages - just have 
- Low latency (< 10 ms)
- Horizontal scaling in consumers
- At least once delivery - duplicates occasionally
- Best effort, but can be our of order
- Max size 256KB/message
- Can set queue to delay (or per message override) up to 15 minutes, default = none
- Not push, consumers poll for messages -> receive up to 10 messages at a time
- Consumer must process messages within visibility timeout
- Consumer delete message with message id and receipt handle

Define messages
- Body up to 256KB string
- Message attributes/metadata
- Delivery delay
-> Return value is a message identifier and an MD5 hash of the body

Visibility timeout
- Message will be invisible to other consumers during the timeout period
- Set between 0 sec and 12 hours (default 30 sec)
- It's a balance: If too high, must wait a long time before processing message agin; if too low, risk duplicated handling
- ChangeMessageVisibility API - change visibility while processing the message (EXAM)

DLQ
- If consumer fails to process in visibility timeout -> goes back to the queue
- "Redrive policy" - how many times a message can go back to the queue
- After that, message goes to DLQ
- You have to create your DLQ first and mark it as a DLQ
- DLQ messages expire too

Long polling (EXAM)
- Normally SQS responds immediately to the poll
- Consumer can also ask for long polling - wait for messages to arrive if there are none
- Wait can be 1-20 seconds
- Recommended to use long polling and use the max wait time (20 sec)
- Enabled at the queue level or API level with WaitTimeSeconds

FIFO queues EXAM
- Deduplication - provide MessageDeduplicationId with your message - deduplication interval is 5 minutes, within that interval other message is ignored
  - Content-based deduplication - automatically calculates the MessageDeduplicationId as SHA256 of body only (not the attributes)
- Sequencing - ensure strict ordering specify same MessageGroupId, e.g. to deliver messages for a user, use the user_id as the MessageGroupId
--> KEY POINT: deduplication always based on group id + deduplication id; deduplication id is calculated in content-based deduplication

SQS EXAM must-know details
- SQS Extended Client - Normally message length limit = 256K, SQS extended client = Java library that uses S3+SQS, sends only small "metadata" message in SQS
- Security
  -- Encryption in transit using HTTPS
  -- Server side encryption (SSE) using KMS - set Customer Master Key (CMK) to use
  -- Data key reuse period (1 min-24 hours, default 5 minutes) - lower = KMS API used more often, more security and more $
  -- Only encrypts body, not attributes
  -- IAM policy must allow usage of SQS
  -- SQS queue access policy - IP limits, time limtis
  -- No VPC endpoint, only thru Internet
- API
  -- CreateQuee, DeleteQueue
  -- PurgeQueue - delete all messages, take up to 60 second
  -- SendMessage, ReceiveMessage, DeleteMessage
  -- ChangeMessageVisibility - change the timeout
  -- Batch APIs for sen, delete, change visibility - reduce costs

SNS - want to send messages to many receivers - pub/sub
- subscribers - SQS, HTTP(s) (with deliver retries), Lambda, email
- integrates with a lot of AWS products - CloudWatch (alarms), ASG notifications, S3 (bucket events), CloudFormation (state changes => failed to build, etc.)
- Topic public - create a topic, create subscription, publish to topic
- Direct publish (advanced) for mobile apps - create platform application, endpoint, public to platform endpoint - works with major mobile publishing services like Google GCM, apple, amazon, etc. - use as mobile app SDK notification service

SNS+SQS = Fanout - publish to many SQS queues
- push once in SNS, several SQS queues subscribe to SNS topic
- fully decoupled, no data loss
- can add receivers of data later
- SQS allows for delayed processing, retries
- many workers on one queue, one worker on anothe queue

Kinesis - popular EXAM topic
- Managed alternative to Kafka
- Great for application logs, metrics, IoT, clickstreams - real time big data
- streaming processing frameworks - Spark, NiFi
- automatically replicated to 3 AZ
- 3 products - used together in a stream
  -- streams
  -- analytics - query using SQL, do real-time computation for fraud detection, etc.
  -- Kinesis firehose - load strings into S3, ElasticSearch, redshift, etc - to store the data somewhere
- streams (=kafka topics) divided into order shards/partitions
- data retention - default 1 day, up to 7 days - massive highway, want to process fast
- ability to reprocess/replay data - with SQS data is gone.
- multiple applications on the same stream
- once data inserted, can't be deleted - immutable - it's a log database
- 1 Mb/s or 1000 messages/sec per shard - 2 MB/s read per shard
- billing is per shard provisioned
- batching or per-message calls
- number of shards evolve over time (reshard/merge) - autoscaling for Kinesis stream
- records ordered per shard like Kafka
- put records + partition key (hashed to determine shard id) - same key always to same partition
- messages get a sequence number, always increasing
- partition key should be high distributed - help prevent "hot partition" - user_id is good key, very distributed, active. Country id is not since many users in same country.
- Batching with PutRecords to reduce costs, increase throughput
- ProvisionedThroughputExceeded to producer if go over the limit
  -- this is per shard, so make sure you don't have a hot shard
  -- can retry with exp backoff or drop
- can use normal consumer library or Kinesis Client Library (KCL)
  -- uses DynamoDB to checkpoint shard offsets, do consumer groups
  -- no more KCL consumers than you have shards
  -- need more consumers -> scale stream up number of shards

Kinesis security
  -- IAM policies to control access/authorization
  -- Encryption in flight using HTTPS
  -- Encryption at rest using KMS
  -- Encrypt client side
  -- VPC endpoints

Kinesis data analytics
- autoscaled
- managed
- continuous/real tieme
- pay for actual consumption rate (not provisioned rate)
- can create streams out of real time queries -> have a stream > use analytics to create a query > output to another stream

Firehose
- Near real time
- Used for ETL, load into redshift / S3 / ElasticSearch / Splunk
- Automatic scaling -pay for amount of data going through Firehose
- Support many data format, pay for conversion

SQS vs SNS vs Kinesis
- SQS - consumers pull data, data deleted after consume, as many workers as want, no provision thruput, no ordering (except FIFO), message delay
- SNS - push data to subs, up to 10M subs, data not persisted, pub/sub, no provision thruput, integrate with SQS for fan-out
- Kinesis - consumers pull data, as many consumers as want, possibility for replay data, meant for real-time big data, analytics, ETL (HINT), order at shard level, temporary data retention, must provision thruput in advance (number of shards)
