# Notifications and integrations

### API Gateway
- Can create CloudWatch log integration: 2 kinds
  - Access logging - similar to access logs in Apache, etc.; can specify fields, or JSON/XML
  - Execution logging - AWS-related logging of each execution
- Access logging can done instead to Kinesis Data Firehose (but not both)
- CloudWatch metrics generated

### ASG
- CloudWatch alarms drive scaling with ALL dynamic policies
  - Simple scaling policy - add/remove N capacity units based on CloudWatch alarm - can have multiple policies (some up, some down)
  - Step scaling policy - specify one CloudWatch alarm w/ multiple steps ("add 1 when 40-70%", "add 2 when 70-90%", "add 3 over 90%")
  - Target tracking scaling - no explicit CloudWatch alarms, add or remove instances to maintain value of some metric like average CPU utilization (creates UP and DOWN alarms behind the scenes, so you can look at history of the alarm in CloudWatch)
- Lifecycle Hooks
  - Hook either on Pending or Terminate
  - Can have a notification target ARN and role (for SQS or SNS)
  - Better choice CloudWatch Event 
    - Service: Autoscaling, EventType: Launch and Terminate
	- KEY IDEA When Lambda is done, needs to call complete-lifecycle-action API call
- CloudWatch metrics
  - ASG related - min/max/desired, number of instances in each state, etc
  - EC2 group metrics - average CPU utilization, disk read/write, etc.
- Notification to SQS or SNS (launch, terminate, fail to launch or terminate) or CloudWatch events - as usual CloudWatch more flexible
- Alarms go to ASG, EC2 actions, SNS notification - alarms are the mechanism an ASG uses to scale
  - Various options (sampling, %, max/min, etc.)
- KEY IDEA No automatic ASG integration with CloudWatch logs - so use CloudWatch agent on EC2's WITH proper IAM role permission

### AWS Config
- Configuration tracking
  - S3 bucket received configuration history - can query e.g. with Athena
  - SNS Notification and CloudWatch events with results
  - SNS notifications - not rule level, across all of AWS Config
    - Use case - want overall operational insights to a slack channel about Config as a whole
    - Per-rule use CloudWatch events
  - Also automatic rule remediation (not as flexible but easier to use and reason about), uses SSM Automations

### CloudFormation
- Specify SNS notifications
- In CloudTrail, only the input parameter key names are logged; no parameter values are logged
  - Presumably because sometimes the values are sensitive

### CloudTrail
- Can also deliver
  - SNS notification for every log file - create automation
  - KEY IDEA CloudTrail to CloudWatch logs - NOTE need an IAM role

### CloudWatch alarms
- Actions - what happens if alarm state
  - Only target = Create one or more SNS notifications
  - Can also create autoscaling action, for an ASG or ECS cluster
  - For EC2 metrics only: also special category of EC2 actions - start, terminate, reboot, etc.
  - KEY IDEA Note: CloudWatch alarms is NOT an event source in CloudWatch Events!
- Billing alarms
  - Only in us-east-1
  - Set threshold normally, send notification when you spend more than (e.g. $10 in a 5 hour period)

### CloudWatch events
- Targets
  - SNS, Lambda
	- Send to a CloudWatch log group
- CloudWatch events integration with CloudTrail
  - Example: Want to know when an EC2 AMI is created. There's no event source for this.
  - However: it does involve making an API call
  - So: use service = EC2, EventType = CloudTrail API call (every service has this option)
    - NOTE Only for read/write APIs such as CreateImage; can't trap on List, Get, Describe API calls
  - KEY IDEA CloudTrail/CloudWatch event integration is great because for just about any modify type API action, you can create an event and handle it

### CloudWatch logs
- CloudWatch logs subscription
  - KEY IDEA Do real-time processing of log events
  - Assigned at the log group level (like Metrics filters, expiration, etc.)
  - KEY IDEA 3 (or 4) basic destinations:
    - Lambda
    - Kinesis streams (not from console, CLI only)
    - Kinesis firehouse (not from console, CLI only)
    - In UI can create to ElasticSearch (creates a Lambda behind the scenes)
  - Kinesis Data Firehose can go to S3, Splunk, ElasticSearch
- All kinds of logs - KEY IDEA to understand options and distinctions
  - Application logs
    - Produced by your application codebuild/MyProjectName
    - Usually on the local filesystem
    - Streamed using a Can agent
    - Direct integration with CloudWatch logs for Lambda, ECS/Fargate/ElasticBeanstalk
  - Operating system logs (event logs, system logs)
    - EC2 or on-prem
    - Inform you of system behavior
    - Also sent via CloudWatch unified agent
  - Access logs
    - List individual requests from people
    - On Load balancers, proxies, web servers
    - Based on user requests being made
  - AWS managed logs
    - Load balancer access logs to S3 - on the ALB level, not target group
    - CloudTrail logs to S3 and CloudWatch logs
    - VPC flow logs to S3 and CloudWatch logs - you can have multiple of these with different log line formats, filters, aggregation time
    - Route 53 access logs only to CloudWatch logs
    - S3 access logs to S3
    - CloudFront access logs to S3

### CloudWatch metrics
- Exporting metrics - S3, elasticsearch, etc.
- Doesn't export metrics natively
- There's an API call get-metric-statistics - get back JSON document with timestamps, max, unit
- Always think: how do I automate this? Create CloudWatch scheduled event -> target = Lambda call 
- KEY IDEA Important metrics - important to know if you need to create a custom metric or automatically available
  - EC2 - CPU, disk, network - but no internal info like memory usage or processes
  - EBS - I/O, queue length - no info on disk space usage
  - ASG - info on ASG itself like Min/max/desired and instances; aggregate on instances
  - ALB - traffic, error rates, connections, capacity
  - RDS - like EC2 but more info because it's managed (memory, disk)

### CodeBuild
- CW Events for failed builds, trigger notifications
- CW Alarms to notify 
- CW Events/Lambda for glue
- SNS notifications with CB trigger
- Logs - Under CodeBuild > Logs - set up log groups - only trace of build after it's finished
  - CloudWatch logs
  - S3
- Triggering builds
  - CLI
  - CodeCommit
    - No direct build source trigger - need to use CloudWatch event/EventBridge
  - S3 - add a bucket and object key
    - No direct build source trigger - need to use CloudWatch event/EventBridge (need CloudTrail integration also)
  - GitHub or GitHub Enterprise - either public repo or connect to your GitHub account
    - Can create a webhook on GitHub to trigger a CodeBuild job
    - Can report status - so you can have GitHub PR's require successful tests
  - BitBucket - similar but no web hook

### CodeCommit
- There are 3 ways you can hook up automation to CodeCommit activities:
  - 1. Notifications - SNS (or AWS Chatbot - e.g. slack) only 
    - literal notification and generally not for taking action based on them (old style)
    - About collaboration
- 2. Triggers - SNS or Lambda - supposed to initiate action, usually some automation
  - About code related events, to branches
  - Can also specify particular branch names (up to 10)
  - Can specify "custom data" such as a chat channel name
  - Events:
    - push to existing branch
    - create branch or tag
    - delete branch or tag
- 3. More flexible way - see blog below - is CloudWatch events (now called EventBridge) - CW is centerpiece of all devops automations
  - Specify a service name - here CodeCommit
  - Then event type - commonly CodeCommit Repository State Change
  - EventBridge also gives you source of any API call via CloudTrail

### CodeDeploy
- CodeDeploy logs - install CodeDeploy agent and CloudWatch log agent - no automatic connection
- Notifications and Triggers - like CodeCommit
- Triggers to SNS only (no lambda unlike CodeCommit)
  - Deployment and Instance triggers - same as CloudWatch state change types
- As with CodeCommit, CloudWatch events/EventBridge are MUCH more flexible (many target types, can specify conditions, multiple targets for an event)
- Integration with CloudWatch alarms (not events!): thousands of types of metrics, per-instance or aggregated.
  - Can use to stop a Deployment
  - Rollback Deployment (if configured)

### CodePipeline
- CloudWatch events together with CodePipeline
  - Auto-created rule tying in the pipeline with the source change
- Already integrated with sources like CodeCommit or GitHub
- Notification rules
  - Detail
  - What are triggers for the notification
    - actions, stages, pipeline
  	- started, succeeded, canceled, resumed, etc.
	  - also for manual approval
- Custom Action provider
  - Invoke Lambda
  - Choose which artifact(s) to send
- Triggering pipeline from other things in AWS
  - CloudWatch/EventBridge event rule
  - Scheduled (e.g. set up a weekly build) or based on source/patterns

### DynamoDB
- Accelerator (DAX) cluster - SNS notifications during maintenance events
- DynamoDB Streams
  - Track write, update, delete
  - KEY IDEA Under the covers, a DynamoDB Streams is a Kinesis Stream - 1MB/s write, 2MB/sec read per shard
  - Can choose only keys, only new data, only old data, both old/new data
  - Hook up to a lambda function
  - If it can't access, lambda IAM role probably doesn't have permissions for DynamoDB
  - Why use? To react in real time to changes
  - NOTE no more than 2 readers (else it throttles) from a stream at once, and global tables also puts a reader on; so if you use Global Tables, you can only have one lambda hooked up
    - So if you want more, have single lambda write to an SNS topic and fan it out to as many as you want
- Outputs CloudWatch metrics such as SystemErrors (API 500 errors), ThrottledRequests, ConsumedReadCapacityUnits or ConsumedWriteCapacityUnits, TimeToLiveDeletedItemCount

### EC2
- Unified CloudWatch Agent
  - Lets you do both metrics and logs
  - 1) Create an EC2 role with the necessary CloudWatch putMetricsData and logs writing permissions, plus SSM
  - 2) Example app: create an apache httpd server, want access_log and error_log logs forwarded to CloudWatch logs
  - 3) Install unified CloudWatch agent - used to be a CloudWatch logs agent and a metrics system and script, etc. - now all in one
  - KEY IDEA monitor host metrics to become custom metrics (per-core CPU, memory, disk) - good because standard CloudWatch metrics (1) are not per-core (2) don't collect memory

### Elastic Beanstalk
- Can do SNS notifications - EB auto-creates and subscribes an email address. 
- Worker environments
  - Two major use cases - a single worker can do both
  - Long running workload (e.g encoding a video) - example program reads SQS queue to pick up work
    - The SQS is created automatically, or can use an external queue (so it's not deleted)
  - Perform scheduled work using a cron.yml file

### GuardDuty
- Pulls three log types (CloudTrail, VPC flow, DNS)
- Continuous, Customizable
  - Integrations: 
    - Notifications and CloudWatch -> Lambda Functions
    - Sends new CWE events within 5 minutes, then update every 6 hours (configurable)

### Health
- Or set up SNS notification with CloudWatch events
  - Health > Specific Health Events 
  - github AWS Health AWS_RISK_CREDENTIALS_EXPOSED
  - Can scan github looking for public IAM keys -> CloudWatch event -> Step functions that invoke lambdas to:

### Inspector
- 1) Make the inspector a target for CloudWatch events
  - the Inspector schedule option does exactly that, a timed event
  - but could do it on some other trigger
- 2) Assessment template has outgoing notification to SNS - CloudWatch did not include Inspector (might now)
- 3) Remediate security findings automatically
  - Install Inspector agent
  - Configure assessment template to output notification to SNS
  - Trigger a Lambda from SNS to do automatic remediation
- 4) Use as the output of a golden AMI pipeline
  - Run as soon as AMI is created by SSM automation
  - Also do timed to make sure AMI is still secure

### S3
- S3 Notification - similar to CodeCommit Notifications in that it does not directly involve CloudWatch events
  - Created on a per-bucket
  - S3 > Properties > Events > Add notification
  - Target can be SNS, SQS, Lambda
  - KEY IDEA For target of SQS you create a policy (like a bucket policy) with a Condition that allows a SourceArn of the bucket
  - Not all actions can be sent with S3 notification - so use a CloudWatch event instead
  - KEY IDEA S3 notifications vs CloudWatch S3 events
    - Notifications always object level (not bucket), only most popular actions, simpler integration
    - CloudWatch event rules bucket or object level, can do almost anything, more complex because goes thru CloudTrail and CloudTrail needs to be enabled for the specific bucket(s) you want

### Service Catalog
- SNS Notifications

### Step functions
- CloudWatch integration
  - Events integration
    - Example: Source = Step Functions, Event = status change ERROR -> target = send a message to slack or call Lambda
    - Example: Source = Scheduled every 1 day -> target = Execute step function state machine
  - Logs integration
    - As of Feb 2020, there is logs integration
    - Can use logs to generate metrics through subscriptions

### Systems Manager (SSM)
- SSM Automation
  - Notifications or CloudWatch events - automate based on automation results
  - Notifications - SNS, CloudWatch, email notification
- Parameter Store
  - Notification of changes with CloudWatch events
  - Automatic references to Secrets Manager

### Trusted Advisor
- Preferences - weekly (no more often) email notifications - separate for billing, operations, and security
- Note that Trusted Advisor is a GLOBAL service - so region = us-east-1
- Notifications
  - CloudWatch events - example rules
  - NO notification to SNS - events only

### OpsWorks
- Q: How do you get notified?
  - Create CloudWatch event, source=OpsWorks > Instance state change -> target=SNS topic, lambda, etc.
  - If you want to know specifically when change due to auto-healing, filter on { detail: { initiatedBy: [ "auto-healing" ]}
  - Or `user` or `auto-scaling`
  - No SNS

### General information on notification and queueing
- SNS - want to send messages to many receivers - pub/sub
  - subscribers - SQS, HTTP(s) (with deliver retries), Lambda, email
  - integrates with a lot of AWS products - CloudWatch (alarms), ASG notifications, S3 (bucket events), CloudFormation (state changes => failed to build, etc.)
  - Topic public - create a topic, create subscription, publish to topic
  - Direct publish (advanced) for mobile apps - create platform application, endpoint, public to platform endpoint - works with major mobile publishing services like Google GCM, apple, amazon, etc. - use as mobile app SDK notification service

- SNS+SQS = Fanout - publish to many SQS queues
  - push once in SNS, several SQS queues subscribe to SNS topic
  - fully decoupled, no data loss
  - can add receivers of data later
  - SQS allows for delayed processing, retries
  - many workers on one queue, one worker on anothe queue

- Kinesis - popular EXAM topic
  - Managed alternative to Kafka
  - Great for application logs, metrics, IoT, clickstreams - real time big data
  - streaming processing frameworks - Spark, NiFi
  - automatically replicated to 3 AZ
  - 3 products - used together in a stream
    - streams
    - analytics - query using SQL, do real-time computation for fraud detection, etc.
    - Kinesis firehose - load strings into S3, ElasticSearch, redshift, etc - to store the data somewhere
