CloudWatch logs
- Log groups - typically an AWS service name+object name, e.g. /aws/codebuild/MyProjectName - contains one or more log streams
- Log streams - e.g. CodeBuild creates with UUID of the build
- Stream contains individual log records
- Log groups by default last forever - at log group level can set the retention period (maybe you're aggregating to S3)

Unified CloudWatch Agent
- Lets you do both metrics and logs
- 1) Create an EC2 role with the necessary CloudWatch putMetricsData and logs writing permissions, plus SSM
- 2) Example app: create an apache httpd server, want access_log and error_log logs forwarded to CloudWatch logs
- 3) Install unified CloudWatch agent - used to be a CloudWatch logs agent and a metrics system and script, etc. - now all in one
- 4) Use SSM parameter store to store the parameters - it's fairly complex
- Supports multiple standard metrics gathering mechanisms
  - StatsD - standard method of generating certain kinds of metrics
  - CollectD - standard log collection daemon
- KEY IDEA monitor host metrics to become custom metrics (per-core CPU, memory, disk) - good because standard CloudWatch metrics (1) are not per-core (2) don't collect memory
  - Add extra metrics dimensions such as imageID, etc.
  - Can generate high resolution metrics
  - Many detailed CPU metrics, disk used/IOPS/reads/writes/time, memory, network interface, processes, swap space
  - Metrics are all defined in the JSON configuration
- Monitor log files by path (e.g. access_log) - goes to this log group and log stream name
- Config stored in local config file /opt/aws/amazon-cloudwatch-agent/bin/config.json
- Better: also store the config file in SSM Parameter Store, so it survives recreating the instance (or use in other instances). Max size 4KB (free) or 8KB (advanced)

So how would you use this to determine not too many 404 errors
- Could create a local custom metrics agent
- Better: create a METRICS FILTER - (1) filters logs (2) create metric(s) (3) create alarms on the metric(s)
  - Define at the log group level - got a funky pattern syntax to match values with wildcard, jq-style JSON matching (since CloudWatch logs are JSON objects)
  - Then create a metric "404NotFound" from the result of the filter
  - Then create an alarm: "404NotFound > 3 for 1 datapoints within 4 minutes"
 
Exporting logs
- Can create export to an S3 bucket - do manually on command
- KEY IDEA - needs permissions - but since it's just plain old CloudWatch, you don't have a role - instead use a bucket policy - principal {"service": "logs.us-east-1.amazonaws.com"}
- KEY IDEA - two ways, either a periodic export command, or create a CloudWatch logs subscription
- You could create a lambda function triggered by CloudWatch Timed Events - something you would have to build
- Has a delay
- A better solution may be a CloudWatch logs subscription

CloudWatch logs subscription
- KEY IDEA Do real-time processing of log events
- Assigned at the log group level (like Metrics filters, expiration, etc.)
- KEY IDEA 3 basic destinations: Lambda, Kinesis streams (not from console, CLI only), Kinesis firehouse (not from console, CLI only)
- In UI can create to ElasticSearch (creates a Lambda behind the scenes)
- Kinesis Data Firehose can go to S3, Splunk, ElasticSearch 
  - Need to give Firehose a role to write wherever it needs to write (S3, etc.)
  - Create a subscription filter from the CLI
  - Subscription filters show up in the UI for the log group

All kinds of logs - KEY IDEA to understand options and distinctions
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
  - List indivdual requests from people
  - On Load balancers, proxies, web servers
  - Based on user requests being made
- AWS managed logs
  - Load balancer access logs to S3
  - CloudTrail logs to S3 and CloudWatch logs
  - VPC flow logs to S3 and CloudWatch logs
  - Route 53 access logs only to CloudWatch logs
  - S3 access logs to S3
  - CloudFront access logs to S3
