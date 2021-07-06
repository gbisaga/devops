Use cases
- For purpose of exam, Lambda is not a general purpose service
- KEY IDEA: Used as glue between other services, to customize how they work (e.g. CloudFormation custom resources)

Basic
- Create and manage service role (BasicExecution role)
  - Execution role has a "trust relationship" with identity provider lambda.amazonaws.com
  - Trust relationship is JSON document like a policy - says service "lambda.amazonaws.com" can do action "sts:AssumeRole"
  - Without that relationship, Lambda functions could not assume the role!
- Always linked to CloudWatch logs - see output of Lambda
- Lambda code - note that it's a small code environment - edit code inline Cloud9 - lambda_function.py is default lambda function file for python
  - Inline no dependencies
  - Upload zip to include dependencies
- Test event - lots of built in event templates e.g. SQS, Kinesis, API Gateway, etc.
- Billed in 100ms increments
- Environment variables (more later)
- Memory - 128MB to 3008MB
  - More memory = more expensive
  - More memory = more CPU, faster execution if CPU bound
  - Could even be cheaper
- KEY IDEA: Timeout - up to 15 minutes - If 1 hour, AWS batch is better; if synchronizing things, maybe AWS step functions
- KEY IDEA: Can put in a VPC 
  - need to do this if function needs to access resources within the VPC (e.g. RDS)
  - can assign a security group - important to access other resources if they use "only in this SG"
- Debugging and error handling
  - Specify a DLQ (SNS or SQS) to deal with events or troubleshoot
  - X-ray integration for active tracing
- Concurrency - maximum number running at once - default limit 1000
- Monitoring
  - CloudWatch metrics - number of invocations, execution duration, error/success counts and rates, throttles, dead letter errors
  - CloudWatch log insights - let you query your logs
  - How many times invoked, and most expensive (is there a bug in the function?)

Triggers - lots of them, DevOps integrations
- API Gateway - let you create external API for serverless application - normally can only invoke thru CLI
- ALB - another external API - simpler HTTP(s) front end
- CloudWatch events - glue of most DevOps integrations - react to any event in the cloud 
  - Cron schedule - serverless cron script
- CloudWatch log - analyze logs in real time, perhaps create alerts
- CodeCommit - look at code e.g. check for credentials being committed
- DynamoDB - streams to react to real time changes in DynamoDB - e.g. users table, react in real time to new users
- Kinesis - real time processing of data
- Async integrations
  - S3 - events when people put objects, create a workflow like create a thumbnail
  - SNS
  - SQS
