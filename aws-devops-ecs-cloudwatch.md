### ECS/CloudWatch logs integration
- CloudWatch logs - a task definition's container definition can have a log configuration
  - can specify different log drivers - awslogs (CloudWatch), splunk
  - auto-configure fills values automatically for group, region, stream prefix
  - no special agent for awslogs - in container (doesn’t take disk space)
  - KEY IDEA: task execution role needs to have log permission
  - can also specify CloudWatch logs agent to send files from the instance's file system
  
### CloudWatch metrics
- Lots of ECS metrics - cluster or service level
- Have memory and CPU metrics built in
- CloudWatch container insights 
  - send per-container metrics into CloudWatch metrics 
  - not just cluster and service, CloudWatch metrics will have one listing for each container
  - can also specify when you create a service

### CloudWatch events
- ECS both an event source OR a target
- Can specify event pattern e.g. ECS > State Change > specific clusters
- Task launched, fails, etc. - same as the events that show up on ECS cluster's Events tab
- Can also use ECS tasks as CloudWatch event target - e.g. can schedule timed event to invoke target ECS task

### CI/CD pipeline with ECS
- ecsworkshop.com/introduction/cicd
  - CodePipeline > use CodeBuild to build docker image to ECR > use CloudFormation to deploy ECS cluster
- KEY IDEA: instead of direct deploys use CodeDeploy to do blue/green to ECS service
- ECR image tags - "latest" is not immutable, will override. Should use either image tag:
  - version tag - 1 to 6, etc
  - use sha256 digest as the image tag
