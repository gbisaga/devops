Trusted Advisor
- Key for automating security and limit events
- Refresh rate, no more often than every 5 minutes
- Two tiers - upper has more recommendations
- 5 categories
  - cost optimization - underutilized EC2s, EBS volumes, idle load balancers
  - performance - non EBS-optimized instances, inefficient DNS configurations, S3->CloudFront recommendations
  - security - bucket permissions, lambdas with deprecated runtimes, IAM key rotation, security groups, exposed access keys in popular public repos (AWS Health has this also)
  - fault tolerance - missing or old EBS snapshots, EC2 distribution across AZs, single AZ RDS
  - service limits - nearing limits (80%) on usage of VPC, Internet Gateway, EIP's, CloudFormation stacks, etc.
- Preferences - weekly (no more often) email notifications - separate for billing, operations, and security

Automations
- Note that Trusted Advisor is a GLOBAL service - so region = us-east-1
- CloudWatch events - example rules
  - Use a lambda -> slack
  - Push notifications to Kinesis
- Check Item Refresh Status - filter on specific statuses, checks, resources
- Lots of examples in https://github.com/aws/Trusted-Advisor-Tools
  - Trusted Advisor finds hi/low utilization 
    -> CloudWatch event 
    -> Lambda to calculate correct instance instance 
    -> Invoke SSM automation document
       - SSM run aws:approve -> SNS -> approver manual approval (similar to CodePipeline but in SSM for AWS automations)
	   - SSM run ResizeAutomation
  - Trusted Advisor sees exposed access keys
    -> CloudWatch event (source: aws.trustedadvisor, check-name: Exposed Access Keys, status: ERROR)
	-> Step Function (1) delete key, (2) CloudTrail to pull uses of key, (3) send to SNS for notification

CloudWatch alarms
- Standard metrics created, like service limits - could create an alarm on it

Automating refresh
- Normally you have to visit Trusted Advisor page - it automatically refreshes once a day there
- Also a manual refresh button on the Trusted Advisor page - can only do once every 5 minutes
- Also an AWS `support` API 
  - `aws support refresh-trusted-advisor-check --check-id XXXX`
  - `aws support describe-trusted-advisor-check-summary`, `results`, `refresh-statuses`
