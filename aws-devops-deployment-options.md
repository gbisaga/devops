# Deployment options

### Elastic Beanstalk - listed by deployment time, least to most
- deploy an application version to an environment 
  - All at once - fastest but downtime - rollback by redeploy
  - Rolling - rollback by redeploy
  - Rolling with Additional Batch - rollback by redeploy
  - Immutable - rollback by redeploy (maybe faster than rolling)
  - Blue/green (Immutable + swap URL) - rollback by swap URL

### OpsWorks
- deploy command and rollback command
- uses deploy recipes
- Blue/green by cloning a stack and switching Route53 (like EB)

### ASG/Load Balancers
- Blue/green by creating a new ALB with new version
  - Can do canary by varying traffic with the Route53 weighted round robin
  - After switch, can leave instances in standby in old group for awhile to verify all is OK
- replace launch Config or template 
  - New instances use new Config
  - can let die, or increase desired then decrease
- also an “instance refresh”
  - not mainly a software redeploy, replaces instances (infrastructure redeploy)
  - does rolling replacements 
  - minimum number healthy
  - can pause for testing and confirmation (notify)

### CodeDeploy
- https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html
- Two basic paths: In-place and blue/green
- In-place
  - EC2 or onprem only (not lambda or ECS)
  - deployment configurations (Both in-place and blue/green support)
    - AllAtOnce - succeeds if any succeed
    - OneAtAAtime - each has to succeed, except the last
    - HalfAtATime
- Blue/green
  - EC2
    - same three options (allatonce, oneatatime, halfatatime)
    - requires ELB, but ASG is optional
    - don’t link multiple CodeDeploy deployment groups to a single ASG (e.g. to deploy multiple independent softwares)
  - ECS all blue green
    - canary, linear, or allatonce
  - Lambda all BG 
    - also canary, linear, or allatonce
- CodeDeploy with ASG
  - https://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-aws-auto-scaling.html

- CodeDeploy "gotchas":
  - A deployment group contains individually tagged Amazon EC2 instances, Amazon EC2 instances in Amazon EC2 Auto Scaling groups, or both.
  - Deployments that use the EC2/On-Premises compute platform manage the way in which traffic is directed to instances by using an in-place or blue/green deployment type. During an in-place deployment, CodeDeploy performs a rolling update (half at a time, etc) across Amazon EC2 instances. During a blue/green deployment, the latest application revision is installed on replacement instances.
  - If you use an EC2/On-Premises compute platform, be aware that blue/green deployments work with Amazon EC2 instances only.
    - You CANNOT use canary, linear, or all-at-once configuration for EC2/On-Premises compute platform (but other BG are ok)
  - You can manage the way in which traffic is shifted to the updated Lambda function versions during deployment by choosing a canary, linear, or all-at-once configuration.
  - You can deploy an Amazon ECS containerized application as a task set. You can manage the way in which traffic is shifted to the updated task set during deployment by choosing a canary, linear, or all-at-once configuration.
  - Amazon ECS blue/green deployments are supported using both CodeDeploy and AWS CloudFormation. For blue/green deployments through AWS CloudFormation, you don't create a CodeDeploy application or deployment group.
  - Your deployable content and the AppSpec file are combined into an archive file (also known as application revision) and then upload it to an Amazon S3 bucket or a GitHub repository. Remember these two locations. AWS Lambda revisions can be stored in Amazon S3 buckets. EC2/On-Premises revisions are stored in Amazon S3 buckets or GitHub repositories.
  - AWS Lambda and Amazon ECS deployments CANNOT use an in-place deployment type.
