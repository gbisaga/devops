### Steps in CI/CD and tools (AWS and non-AWS):
- 1) Code - CodeCommit, GitHub, 3rd party
- 2) Build - CodeBuild, Jenkins CI
- 3) Test - CodeBuild, Jenkins CI
- 4) Deploy - ElasticBeanstalk, CloudFormation, CodeDeploy, ECS 
- 5) Provision - ElasticBeanstalk, CloudFormation, CodeDeploy
- To orchestrate the whole thing in AWS: CodePipeline (or Jenkins)

### CodeCommit - TO KNOW
- Summary
  - Benefits: collaboration, code is backed up, fully viewable/reviewable, auditable
  - Only private repos - not public
  - No size limits, scale seamlessly
  - User interactions all using standard git
- Security 
  - Authentication: SSH keys, HTTPS, MFA
  - Authorization: IAM policies
  - Encryption at rest: automatic, using KMS
  - Cross account access: IAM role + STS AssumeRole
- Notifications - CloudWatch events
  - About collaboration
  - Full or Basic information
  - Events: comments on commits or PRs; PR CRUD; branch/tag CRUD
  - Only to SNS topic
  - use to set up pipeline for feature branch builds
- Triggers
  - About code related events
  - Push code to branch, create/delete branches
  - Either to SNS or Lambda

### CodePipeline
- Orchestration - Visual workflow CD tool
- Build with CodeBuild/Jenkins/etc
- Load testing with 3rd party tools
- Deploy with AWS CodeDeploy, Beanstalk, CloudFormation, ECS...
- Multiple stages with sequential or parallel actions
  - Manual approvals can be used at any stage
  - Each stage can have artifacts into S3
  - Each stage has one or more Action Groups (Pipeline > Stage > Action Group > Action) 

### Troubleshooting - important for Exam
- CodePipeline state changes happen in CloudWatch events, which can create SNS notifications. Ex: Failed pipelines, canceled stages.
- If CodePipeline fails astage, pipeline stops. Get notification, look in console.
- CloudTrail can be used to audit API calls
- If CodePipeline can't perform an action, make sure the "IAM Service Role" attached to pipeline has enough permissions (IAM Policy)

### CodeBuild
- Managed build service - alternative to others such as Jenkins.
- Continuous scaling - no build queue
- Pay only for usage of build servers
- Uses docker for reproducible builds
- Can extend using instead of standard Amazon image, use our own custom base images
- Secure: KMS for artifact encryption, IAM, VPC, CloudTrail for API call logging
- Source code from many places (GitHub, CodeCommit, CodePipeline, S3...)
- Build instructions in code (buildspec.yml file - EXAM) - buildspec.yml file - normally in root directory. However you can change the name (for multiple builds in a single project) or file location.
- Output logs to S3 and CloudWatch Logs - container that ran build is gone (so you can't remote into them, etc.), but logs live on
- Metric, CloudWatch Alarms to detect failed builds
- CloudWatch Events/Lambda as a Glue
- Define builds in CodePipeline or CodeBuild (confusing)
- Built-in lots of standard language support environments (java, ruby, python, node, go, android, .net core, PHP), or can add your own docker image to build
- Source code (CodeCommit)  +  Build Docker image (prebuilt or custom)
                            V
        CodeBuild container + optional S3 cache bucket
                            V
 artifacts to S3 bucket + logs in CloudWatch or S3 + S3 cache if used
- In buildspec.yml file:
  - environment (plaintext or secrets ref in SSM Parameter Store) 
  - 4 Phases: install, pre-build, build, post-build (e.g. create a zip file)
  - each has a finally block
- Artifacts: what to upload to S3 - always encrypted with KMS
- Cache: files to cache for future build speedup (usually dependencies)
- Can reproduce and troubleshoot builds locally - install docker and CodeBuild Agent
- Deploy an indivdual build in a VPC - in case the build needs access to the VPC's resources

### CodeDeploy
- Deploying code automatically to EC2 instances 
- Can use instead of ansible, terraform, chef, etc - but it's managed
- Most important Exam thing: 
  - Each machine must run CodeDeploy Agent
  - Can be EC2 or on-prem - Specify with Tags
  - Need appspec.yml at root of project
- Instances groups by deployment group (dev/test/prod etc. - choose names yourself)
- Existing setup tools, works with any application, autoscaling integration
- Blue/green only works with EC2
- Has support for AWS Lambda
- CodeDeploy only deploys, doesn't provision resources
- Deployment type (blue/green or in-place) and group

### appspec.yml file - two parts - MOST IMPORTANT for exam, order is logical and important
- File section: how to source and copy
- Hooks:
  - ApplicationStop - 
  - DownloadBundle
  - BeforeInstall
  - Install
  - AfterInstall
  - ApplicationStart
  - ValidateService - REALLY IMPORTANT - the health check, makes sure it deployed correctly

### To Know:
- Configs 
  - Deploys - one at a time/all at once/half at a time, min healthy host %
  - Failures - stay in failed state or rollbacks
  - Deployment targets - EC2 instance or ASG or Lambda
- Deployment models
  - In place all or 50% at a time
  - Blue/green - like immutable
- 2 Roles: 
  1) CodeDeploy service role - Use case "CodeDeploy" with lots of policies
  2) EC2 service Role - needs at least S3 read access to pull artifacts 
- Deployment group - bunch of tagged instances or ASG/ECS + Deployment

### CodeStar
- Integrated solution that wraps all the CI/CD pieces together - one dashboard to view all components, even CloudWatch metrics
- Quickly create CICD ready projects for EC2, Lambda, Beanstalk
- Integration with jira/GitHub issues
- Cloud9 integration
- Free, simple with limited customization
