Jenkins on AWS - white paper https://d1.awsstatic.com/whitepapers/DevOps/Jenkins_on_AWS.pdf
- KEY IDEA: You can set this all up yourself, or just use CodePipeline for a serverless managed solution
- Open Source CI/CD tool
- Can replace CodeBuild, CodePipeline, or CodeDeploy - all or part of them - can use together
- Must be master/slave configuration
- Difficult - multi-AZ master, deploy on EC2'slave
- All projects need a Jenkinsfile (like buildspec.yml)
- Lots of plugins
- Three options:
  - Master and workers are on the same instance
  - Master separate from your workers - much more scalable
  - Use master/multi master with CodeBuild plugin - HA, less setup (no slave infrastructure)
- Master would be an EC2, with slaves in others, maybe in an ASG
- Can have one master, or a multi-master setup - multi-masters share info using EFS

KEY IDEA Differences with CodePipeline, how to choose
- CodePipeline
  - No managing server, just use the service (don't care about versions, OS, etc.)
  - Built-in actions provided by AWS (source, test, etc) - make easier to interact with AWS services
  - Can write custom actions
  - IAM based authentication - manage same as other IAM-using services
- Jenkins
  - Self-managed infrastructure - master coordinates, slaves do the work like CodeBuild
  - Lots of operational work
  - Plugins - similar to CodePipeline actions
  - You also have total control over OS or manual patching, running on-premise
  
Lots of Options
- Use Jenkins as part of a stage in CodePipeline - replaces CodeBuild, take advantage of all the plugins
- Use Jenkins instead of CodePipeline - Jenkins runs the whole pipeline: control (CodePipeline), build (CodeBuild), deploy (CodeDeploy)
- Use Jenkins to directly call Device Farm (test mobile app)
- Jenkins to deploy Lambda
- Jenkins with CloudFormation - pulls CF templates with security review, then invokes CloudFormation

Set up Jenkins
- Install on an EC2 - to do simply just create a single EC2 for master and slave (or "agent") together
- Use a key from a file to do initial authentication - create a user for thereafter
- Then install recommended plugins and lots of others - there's a CodePipeline plugin

KEY Plugins you need to know
- Plugins to manage agent/slaves on demand and only pay for what you use
  - EC2 plugin - allow to automatically create EC2 instances and stop when unused - manage fleet of slave/agents
  - AWS CodeBuild - actually uses CodeBuild instead of slave instances - don't need any slave instances! More serverless
  - ECS or ECS with autoscaling - launch agent/slaves into ECS
- CodePipeline - to use Jenkins in CodePipeline
- S3 Artifact Manager - allows Jenkins to keep artifacts in S3
