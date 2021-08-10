### DynamoDB - Feature overview
- Create a table
  - KEY IDEA Need at least a unique primary key - the partition key - choose carefully. Great overview of it https://aws.amazon.com/blogs/database/choosing-the-right-dynamodb-partition-key/
  - Consider if also want a sort key (aka index)
  - NOTE partition key plus sort key = primary key for the table (can't change this for a record, only make a new record)
  - All other attributes define at runtime - not predefined
  - Local Secondary Indexes (LSI) - need to be defined at table def time - cannot add later (Global Secondary Indexes you can add later)  
    - This is like a materialized view of the table
    - Why want them? If you want more than one sort key/index equivalent.
    - Can only define as an LSI if the partition key of the index is the SAME as partition key of the table itself
	- Also define attributes to project to the LSI (All, Keys, or Select specific attributes)
  - Global Secondary Indexes - can add later
    - Any field as the partition key and sort key
	- Also have projection
  - Capacity of table - two choices
    - Provisioned (free tier elegible)
      - By default you have to provision capacity for your Table (LSIs inherit) and GSIs (separately)
	  - Number of RCUs and WCUs
	  - Can also choose autoscaling of provisioned capacity
	    - Separate autoscaling for RCU and WCU - optional separately for each
		- Can also choose to autoscale capacity in GSIs
		- Min/Max/Target utilization
	- Also have on-demand option
	  - As much as workload demands
	  - A lot more expensive than provisioned
  - Always encrypted at rest - choose default or specify KMS key
- DynamoDB Accelerator (DAX) cluster
  - Cache in front of table
  - Choose instance size, cluster size (>3 for production)
  - Optional encryption
  - Needs an IAM role to access DynamoDB service
  - In a VPC
  - Settings like TTL, AZ preference, notifications, maintenance windows, etc.

### DynamoDB Streams
- Track write, update, delete
- KEY IDEA Under the covers, a DynamoDB Streams is a Kinesis Stream - 1MB/s write, 2MB/sec read per shard
- Can choose only keys, only new data, only old data, both old/new data
- Hook up to a lambda function
- If it can't access, lambda IAM role probably doesn't have permissions for DynamoDB
- Why use? To react in real time to changes
- NOTE no more than 2 readers (else it throttles) from a stream at once, and global tables also puts a reader on; so if you use Global Tables, you can only have one lambda hooked up
  - So if you want more, have single lambda write to an SNS topic and fan it out to as many as you want

### “Global” Tables
- Replicate a table in another region
- Don't have option to create a global table until you add a Stream to the table
- Add as many regions as you want - great for low latency access in multiple reasons
- This is two-way replication between regions
- Table has to be empty before you enable multiple regions

### TTL
- Create a TTL attribute (can be named anything you want)
- Define that attribute as the table's TTL (seconds past 1970)
- At time, deletes entry, sends a DELETE to the stream, forwards to other regions if global table

### Two common patterns for DynamoDB
- Build metadata index for S3
  - Write a lot of items to S3 -> lambda -> puts metadata into DynamoDB table
  - Create an API for object metadata
    - Search by date, total storage for a customer, list of all objects with attribute or by date
- Hook up to Amazon ElasticSearch to get a better search capability
   - DynamoDB -> stream -> lambda -> update ElasticSearch
