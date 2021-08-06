Monitoring - users only care if application is working
- Is latency good? Will it increase over time?
- Outages - experience shouldn't be degradad
- Find out problems before users know
- Can we prevent issues before they happen?
--Monitoring Services
  - CloudWatch - metrics, logs, event, alarms
  - XRay - troubleshoot performance and errors, distributed tracing and performance
  - CloudTrail - monitor API calls, audit changes to AWS resources by users

Metrics
- Provides metrics for all services in AWS - metric = variable to monitor
- Grouped by namespaces (e.g. EC2, Auto Scaling, ElasticBeanstalk - shown in starting Metrics page of CloudWatch)
- Dimension is an attribute, up to 10 dimensions/metric, metrics have timestamps
- Can create dashboard
- By default metrics every 5 minutes; can have detailed monitoring (extra cost) every 1 minutes
  - reason - more quickly scale ASG
- EC2 instance MEMORY USAGE by default NOT pushed as a metric - need a custom metric
- Custom metric resolution = StorageResolution parameter - 1 minute by default, High Resolution down to 1 second (extra cost)
- Use API PutMetricData
- Use exponential backoff ofr throttle errors
- EC2 disk metrics - only instance store. EBS metrics on the volume, not the instance

Alarms
- Trigger notifications for any metric
- Alarms go to ASG, EC2 actions, SNS notification - alarms are the mechanism an ASG uses to scale
- Various options (sampling, %, max/min, etc.)
- On high resolution metric can trigger every 10 seconds
- Alarm states: OK, INSUFFICIENT_DATA, ALARM
- Period - length of time
- Example: used by ASGs. Puts in two alarms 
  - if > max limit, add a CPU; this is normally in green OK state
  - if < min limit, remove a CPU if > 1; this is normally in red ALARM state if nothing is happening
  
Logs
- Sends log using the SDK
- Can collect from any places -EB, ECS (containers), log agents on any EC2 (include onprem), Lambda, VPC flow logs, API gateway, CloudTrail, Rt 53, etc
- Can go to S3 or stream to ElasticSearch for analysis
- Filter expresssions
- Architecutre Log Groups (representing Application) > Log stream (instances or containers, sometimes identified by UUID) > Log records
- Can define log expiration policies
- Can tail logs with AWS CLI
- IMPORTANT FOR EXAM: To send, make sure IAM permissions are correct!
- At log group (application) level you can
  - Set to encrypt logs using KMS
  - Export to S3
  - Stream to Lambda or ElasticSearch
  - Set expiration policy - Never expire by default but can be expired e.g. 7 days
- ElasticBeanstalk can maintain configuration to log to CloudWatch - e.g. access logs of NGINX, environment health, nodeJS log, etc.

CloudWatch events - kick off targets - two options
- 1) On a schedule: fixed rate or cron job type specification
- 2) event patterns, rules to react - pair of (service name, event type, conditions) 
     example: (CodePipeline, Execution State change, states=[Failed])
- Set one or more targets: write to SNS topic, Kinesis stream, SQS queue, call Lambda, etc.
- Often created automatically e.g. by CodeCommit, CodePipeline
