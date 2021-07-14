CloudTrail - log all API events.
- Remember most everything in AWS uses APIs
- Create a trail of (1) API calls (2) S3 accesses (3) Lambda invocations
- KEY IDEA Can deliver trail from all regions or only current region
- KEY IDEA Can receive log files from multiple accounts
  - Have a central account to collect the CloudTrail log files
  - Change the S3 bucket policy to allow resource from multiple accounts
  - Could include account number in to limit access to only their own logs
- Filter trail by Read, write, etc
- Can encrypt logs - by default uses SSE-S3 encryption, can use KMS instead
- Can validate
- Can also deliver
  - SNS notification for every log file - create automation
  - KEY IDEA CloudTrail to CloudWatch logs - NOTE need an IAM role

KEY IDEA What's inside a CloudTrail file?
- JSON file - an array of Record
- userIdentity - who made the request - e.g assumed role by ECS scheduler
- event information - eventTime, Source (elasticloadbalancing.amazonaws.com), resources
- requestParameters - look at such and such an ARN
- responseElements
- who, when/where, what was requested, response(s)
- not in real time (use CloudWatch events - has the exact same JSON structure), delayed for a period
- Can search in CloudTrail for events with name, by whom, etc.

What are these used for?
- Security event in the application
- Q: Can hacker change the log files?
- KEY IDEA AWS CLI has CloudTrail log file validation `aws cloudtrail validate-logs`
  - Works by comparing a digest file - note that it only gets delivered once an hour
