### CloudWatch events - KEY IDEA - the centerpiece of our DevOps integrations
- New service called EventBridge - basically the same as CloudWatch events, plus some extras like external or custom/partner events (e.g. Datadog, PagerDuty)
- Event rules
  - Source have two options
    - Scheduled - cron expression
	  - Typically go to a Lambda as a target
    - Event pattern - look at AWS service and look for patterns
	  - Service name - nearly every AWS service - for us it's the pipelines, etc. - think about all possible service types to integrate
	  - Event type
	  - States
	  - Can directly use an Event Pattern JSON document
  - Targets (one or more) - lots of common integrations
    - Popular is Lambda - you can do anything
	- Several EC2-specific target API calls such as CreateSnapshot, Reboot/Terminate/StopInstances
	- Launch a batch job
	- Send to a CloudWatch log group
	- Launch ECS task
	- Send to Firehose/Kinesis
	- SNS/SQS
	- Step function step
	- SSM to run remote commands, etc.

### CloudWatch events integration with CloudTrail
- Example: Want to know when an EC2 AMI is created. There's no event source for this.
- However: it does involve making an API call
- So: use service = EC2, EventType = CloudTrail API call (every service has this option)
  - NOTE Only for read/write APIs such as CreateImage; can't trap on List, Get, Describe API calls
- KEY IDEA CloudTrail/CloudWatch event integration is great because for just about any modify type API action, you can create an event and handle it

### S3 Notification - similar to CodeCommit Notifications in that it does not directly involve CloudWatch events
- Created on a per-bucket
- S3 > Properties > Events > Add notification
- Target can be SNS, SQS, Lambda
- KEY IDEA For target of SQS you create a policy (like a bucket policy) with a Condition that allows a SourceArn of the bucket
- Not all actions can be sent with S3 notification - so use a CloudWatch event instead
- KEY IDEA S3 notifications vs CloudWatch S3 events
  - Notifications always object level (not bucket), only most popular actions, simpler integration
  - CloudWatch event rules bucket or object level, can do almost anything, more complex because goes thru CloudTrail and CloudTrail needs to be enabled for the specific bucket(s) you want

### Dashboards
- Add widgets - line. stacked area, number, text, etc
- Choose metrics for widgets, customizable
- KEY IDEA Dashboards can be multi-region and multi-account so you can mix regions/accounts in a single dashboard
  - You select the region when creating data items in a widget
- KEY IDEA "Correlate" data in a single dashboard - watch for this keyword, it means use a CloudWatch dashboard
- Can create automatic dashboards with a click, aggregates most common metrics
