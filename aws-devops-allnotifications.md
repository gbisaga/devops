Notifications and integrations
### ASG
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

### CodeBuild
- CW Events for failed builds, trigger notifications
- CW Alarms to notify 
- CW Events/Lambda for glue
- SNS notifications with CB trigger

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
- Notifications and Triggers - like CodeCommit
- Triggers to SNS only (no lambda unlike CodeCommit)
  - Deployment and Instance triggers - same as CloudWatch state change types
- As with CodeCommit, CloudWatch events/EventBridge are MUCH more flexible (many target types, can specify conditions, multiple targets for an event)

### CodePipeline
- Notification rules
  - Detail
  - What are triggers for the notification
    - actions, stages, pipeline
  	- started, succeeded, canceled, resumed, etc.
	  - also for manual approval

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

### Elastic Beanstalk
- Can do SNS notifications - EB auto-creates and subscribes an email address. 

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

### Service Catalog
- SNS Notifications

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
- CloudWatch events - example rules

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
