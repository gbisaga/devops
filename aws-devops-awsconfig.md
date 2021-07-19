AWS config - paid service, extremely expensive
- Provides audit trail on compliance of resources
- Watches configuration and compliance over time
- Can add customized rules, but rules are optional - on its own, it tracks configuration changes in the resource
- 1. Configuration tracking
  - S3 bucket received configuration history - can query e.g. with Athena
  - Notification and CloudWatch events with results
  - KEY IDEA Needs an IAM role and also a bucket policy to allow config service to do bucket check and delivery
  - AWS Config discovers all resources - instances, roles, SGs, buckets, stacks, etc.
  - Also lists CloudTrail events for that resource (config reads CloudTrail)
  - Shows exact configuration change, e.g. t2.micro -> t2.small (even if not running)
  - KEY IDEA Configuration timeline, relationships (who changed when), export, S3 bucket
- 2. Compliance tracking via rules
  - Create set of rules with compliance trail, dashboards
  - Each rule costs money just to define ($1/month)
  - Check e.g. if HTTP-to-HTTPS redirection on LBs, check SGs for unrestricted IP addresses
  - Rules can be triggered periodically or on changes to resource
  - Can specify a remediation action
  - Shows all the compliant and non-compliant resources
  - Also custom rules 
    - run a Lambda function
	- trigger types are important - configuration changes or periodically
	- then specify the scope - what resources you looking at: specific tags, specific resources or resource types - so checking tags on all resources, checking config of only EC2, etc.
	- extra parameters to pass to the Lambda
	
AWS Config Automations
- SNS notifications - not rule level, across all of AWS Config
  - Use case - want overall operational insights to a slack channel about Config as a whole
- Per-rule use CloudWatch events
  - KEY IDEA CloudWatch events are the centerpiece of all automation
  - CloudWatch source=event > Config > Config rule compliance change > Specify type and rule name(s) > Specify resource types or ids
  - Target = Lambda, start Step function, SSM automation, EC2 action like terminating an instance
  - Rule remediation (not as flexible but easier to use and reason about), uses SSM Automations

AWS Config can monitor across multi-account, multi-region aggregator, or organization
- Aggregated View > Create aggregator (in a master account) > Source accounts (ids or organization) > Regions
- Needs to authorize aggregation in each account/region
