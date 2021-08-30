IAM best practices
- Never use the root account, enable MFA for root
- Never grant a policy with "*" access to a service
- Monitor API calls made by a user in CloudTrail
- Never store IAM credentials anywhere but personal or onprem server
- EC2 and Lambda and ECS tasks (ECS_ENABLE_TASK_IAM_ROLE=true), CodeBuild should each have their own roles 
  - don't re-use roles
  - Create least-privilege roles
- Cross-account access - (1) create special IAM role (2) define which accounts can access it (3) Use STS and the AssumeRole API to get a temporary credentials (good for 15-60min)

### Advanced IAM (EXAM)
- Authorization model, evaluation of policies, simplified
  1. If explicit DENY (users can't create DynamoDB tables), then DENY - NOTE that explicit DENY wins!
  2. If an ALLOW, then ALLOW
  3. Else DENY (i.e. the decision starts with DENY)
- S3 bucket policies and IAM policies - the UNION of the two policies is used. But the DENY still wins
- Dynamic policies - how to give each user a /home/<user> flder in an S3 bucket? Don't want per-user policies, not scalable.
  - Use special policy variable ${aws:username}
  - Use variable in the resource ARN
- Inline vs managed policies
  - AWS Managed policies - good for Power users and administrators
  - Customer Managed Policies - re-usable, version controlled with rollback
  - Inline: strict one-to-one relationship between policy and principal
  - can use both (managed gives ALLOW, customer gives DENY)
