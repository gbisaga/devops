Multi-account - Organizations
- Organizations is global service, multiple accounts in the organization
- Master and member accounts
  - One master account - can't change it
  - Member accounts - can only be part on one organization, but can be migrated from one organization to another
- Consolidated billing across accounts, since payment method
- Pricing benefits from aggregated usage - volume discount across all accounts
- API to automate account creation
- Organizations can be full (more modern) or only will billing controls (older)
- Organization can see IAM accesses (including last-used info) - only from organization's master account
Multi-account strategies - different options
- Create multiple Accounts: per department, cost center, dev/test/prod, separate per-account service limits, isolated accounts for logging
- Another option: single account with multi-VPC
  - Not separate users, have access to all resources
- Tagging standards for billing
- Enable CloudTrail or CloudWatch logs being sent to central S3 account
- Establish cross-account roles for admin purposes

Organizational units (OUs)
- Organize accounts into a tree/hierarchy directory-like structure
- Master account at the root OU + 3 OUs (sales, retail, finance) which have several accounts each
- Define OUs by business unit, environment, project-based
- OUs contain accounts plus other OUs
- OUs inherit some rules from parents

Service Control Policy
- Whitelist or blacklist IAM actions
  - Ex: nobody in the account can create an EC2 instance
  - Applies to users and role of the accounts, including root or admin users - NOT applied to Master Account
  - Does not affect service-linked roles (roles that other services use)
  - Explicit DENY takes precedence over ALLOW
- KEY IDEA Use cases
  - Restrict access to certain services (e.g. can't use EMR)
  - Enforce PCI compliance - disable certain services
- KEY IDEA Example in an organization hierarchy - understand this
  - Structure:
    - Root OU - KEY IDEA Give Root OU - FullAWSAccess - SCP needs explicit allow (by default doesn't allow anything)
	  - Master account - apply DenyAccessAthena SCP -> useless because this is the master account and no SCP applies
	- Prod OU - Apply DenyRedShift -> will be inherited by children
	  - Account A - Apply AuthorizeRedShift -> inherits DenyRedShift; has no effect since DENY takes precedence -> So ALLOW policies only useful if Root OU is not FullAWSAccess
	  - HR OU - Apply DenyAWSLambda
	    - Account B - inherits DenyRedShift and DenyAWSLambda
	  - Finance OU
	    - Account 3 - inherits DenyRedShift but NOT DenyAWSLambda - same SCP as Account A
- Blacklist/whitelist strategies
  - Looks like a policy JSON from IAM, bucket policies, etc. (RAPE elements - not sure about Principal)

Migrate accounts from one organization to another
- Remove organization from old account
- Send an invite for the new organization to the member account
- Accept the invite to the organization in the member account
- To migate the master it's the same, but you need to migrate all the member accounts first and then delete the old organization
- Can create accounts directly in an organization

Multi-account service integration (trust)
- Any cross-account action needs IAM "trust" relationships - action to other AWS accounts
- IAM roles assumed cross-account
  - Never share IAM creds
  - Use AWS Security Token Service (STS)
- CodePipeline
  - Cross account invocation (not thru console)
  - Deploy in multiple accounts
- AWS Config Aggregators (multi-region, multi-account, across entire organization)
- CloudWatch events -> create an event bus, can read across accounts
- CloudFormation StackSets - multi-region, multi-account
- Centralize logs with automation!
  - First create CloudWatch log destination
    - Log destinations are part of Landing Zone, which is part of Control Tower https://aws.amazon.com/solutions/implementations/aws-landing-zone/
    - Use CLI - MUST BE IN MANAGEMENT ACCOUNT OF ORGANIZATION -> subscription to Kinesis Firehose -> S3
  - CloudWatch log group create event -> Lambda to create new log group subscription
  - Connect log group to the other account log destination
  - CloudWatch logs are delivered to the central logging account
