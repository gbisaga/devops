# Deployment options

### Elastic Beanstalk - listed by deployment time, least to most
- deploy an application version to an environment 
  - All at once - fastest but downtime - rollback by redeploy
  - Rolling - rollback by redeploy
  - Rolling with Additional Batch - rollback by redeploy
  - Immutable - rollback by redeploy
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
  - ECS
    - canary, linear, or allatonce
  - Lambda
    - also canary, linear, or allatonce
- CodeDeploy with ASG
  - https://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-aws-auto-scaling.html


