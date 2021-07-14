Kinesis - similar but different API to Kafka
- Automatically replicated (highly available) to 3 AZs
- 3 major services in Kinesis
  - Streams - low latency streaming
  - Analytics - real time analytics
  - Firehose - load streams to S3
- Typical - use Streams -> Analytics to do some analysis -> Firehose to go to a destination, like S3
- Divided into shards (partitions) - producers distribute into shards
- Data retention 1 day by default - replay data or multiple consumers
- Scales
- Data is immutable once put in Kinesis
- Billing is per shard provisioned
- Can evolve number of shards over time
- Records orderd per shard, but not across shards
- Records
  - Data blob, up to 1 MB
  - Record key -> to direct to a shard - can have a "hot partition" problem
- KEY IDEA limits - Producer 1 MB/s (or ProvisionedThroughputException) - Throttling in consumers 2 MB/shard - retention 24 hrs (default) to 7 days
- Many kinds of producers can send data into Kinesis
  - Kinesis SDK
  - Kinesis producer library (KPL)
  - Kinesis agent
  - CloudWatch logs
  - 3rd party libraries: spark, log4j appenders, flume, etc.
- Many kinds of consumers: SDK, Kinesis Client Library (KCL), firehose, Lambda
  - KEY IDEA KCL is most important: uses DynamoDB to checkpoint offsets
  - Great for reading in distributed manner
  - Note that this is like the high level consumers in Kafka, but it uses DynamoDB to store offsets automatically
  - Only one KCL application per shard

KEY IDEA Kinesis Firehose (aka a "delivery stream" vs a "data stream")
- near real time (60 second)
- KEY IDEA deliver data to Redshift, S3, ElasticSearch, Splunk
- automatic scaling
- data transformation thru Lambda (ex: CSV -> JSON)
- compression when target is S3
- pay for amount of data going thru Firehose
- KEY IDEA can go into it with many sources, but most important are Kinesis Data Streams and CloudWatch logs and events
- KEY IDEA Streams vs Firehose - must understand the difference between them
  - Streams is you write custom code to consume, real time, you must manage scaling, data storage for 1-7 days
  - Firehose is fully managed, to S3/Splunk/Redshift/ElasticSearch, near real time (>= 1 min), autoscaling, no data storage

Kinesis Analytics - high level analysis using SQL
- Managed, autoscaling, real time
- Create streams from real-time queries
