CodePipeline is the definition of your CI and CD
- Best practices and use cases: https://docs.aws.amazon.com/codepipeline/latest/userguide/best-practices.html - understand at high level
  Read shows how to use all together: https://aws.amazon.com/blogs/devops/implementing-gitflow-using-aws-codepipeline-aws-codecommit-aws-codebuild-and-aws-codedeploy/
- Used to automate the whole flow from source to build to deployment
- Visual workflow
- Sources: GitHub, CodeCommit, S3
- Build tools: CodeBuild, Jenkins, etc
- Load testing: 3rd party tools
- Deploy: CodeDeploy, Beanstalk, CloudFormation, ECS, ...
- Stages
  - Sequential actions hooked together
  - Can have approval steps
  - Each stage can create artifacts, passed on via S3
  - Trigger starts > Source (CodeCommit) > Build (CodeBuild) > Deploy (CodeDeploy) - artifacts between Each

Key principles of CI/CD
- Automation - do the boring stuff, make sure it works in a "clean" environment
- Everything as code (no manual adjustments) - including the pipeline itself
- Test, test, test
- Consistency
- Integrate frequently - a lot safer, smaller sets of changes - finding the issue 

Pipeline
- KEY IDEA #1: Have to specify a repo and branch, so need one pipeline for each branch
- KEY IDEA #2: Detection option, CloudWatch events (recommended, because it happens immediately) or CodePipeline to check periodically.
  - Creates a CloudWatch event rule for the purpose, target is the CodePipeline pipeline with the service role created. Rule is e.g.
    {
        "source": [
            "aws.codecommit"
        ],
        "detail-type": [
            "CodeCommit Repository State Change"
        ],
        "resources": [
            "arn:aws:codecommit:us-east-1:356371446905:garytest1"
        ],
        "detail": {
            "event": [
                "referenceCreated",
                "referenceUpdated"
            ],
            "referenceType": [
                "branch"
            ],
            "referenceName": [
                "master"
            ]
        }
    }
- Creates a service role
- Each pipeline starts with specific predefined stages (not as flexible as gitlab, but after creating you can add as many stages as you want with the visual editor). Must have at least two stages, with EITHER a build or deployment stage (or both)
  - Source stage - pull code
  - Build stage - optional, can skip if no build activity is required
  - Deploy stage - deploy e.g. with CodeDeploy
- Source stage
  - Choose a location (S3) - "default location" creates a new one for each pipeline, will have many created; or specify one you have
  - Default or customer managed KMS key
  - Choose a source - ECR (trigger pipeline when a new docker image published), CodeCommit (trigger on commit), GitHub, BitBucket, etc.
  - Repo and a specific branch - pipeline per branch
  - Choose detection option - CloudWatch events recommended
- Build stage - optional - want to use for more safety
  - Service role created, but need to make sure this can also access our own S3 bucket where the code is
- Deploy stage - use a provider - many like CodeDeploy, CloudFormation, Beanstalk, Alexa, ECS
  - Note that you can deploy into a different region - specify the region(s) and trigger deploys into those regions from one pipeline

Once created, the pipeline is "running" (i.e. waiting to be triggered). You can stop it from running or edit it.
- Edit, add more stages
- Each stage has action groups
  - Each action group has one or more actions
  - Note that action groups are executed SERIALLY while actions within a group are executed IN PARALLEL
  - In code, you specify a runOrder https://docs.aws.amazon.com/codepipeline/latest/userguide/reference-pipeline-structure.html - parallel = same runOrder; this is how you specify Sequential vs parallel actions in code
  - Actions have a provider - many possible providers, grouped into types. Some providers in more than one type (e.g. CodeBuild is both build and test)
    - Approval
    - Build
    - Deploy
    - Invoke - run anything you want (e.g. via Lambda)
    - Source
    - Test
- Each stage specifies one or more artifacts
- Save and "Release Change" - this causes pipeline to re-trigger

CodePipeline and S3 - KEY IDEAS
- CodePipeline uses S3 constantly for reading and writing artifacts
- #1 Per-pipeline (default) or central
- #2 Always encrypted, but can either use account default or a customer managed KMS key
- #3 At every stage of the pipeline, we create artifacts in the S3 folder; either standard or custom names
- #4 Also concept of CodeBuild artifacts - separate from CodePipeline artifacts
- Can also upload to other S3 buckets - create an action to upload to desired location

Manual Approval steps
- Add ActionGroup before the deploy itself (need to be a group because need it to be Sequential, not parallel)

All integrations - high level use cases - read best practices document
- Manual approval - want review
- CodeBuild - build or test code
- Jenkins - integrate with 3rd party build service
- CloudFormation - deploy infrastructure
- CodeDeploy
- Elastic Beanstalk - deploy there, automatic rolling upgrades
- Service Catalog
- Alexa skills
- ECS - simple or blue/green
- S3 - upload artifacts to another region e.g.
- Lambda - you can do anything here!
- CodeCommit, ECR, S3, GitHub - pulling code to build
- AWS Device Farm - test on physical devices, load testing
- BlazeMeter, Ghost inspector, RunScope - UI, API, performance testing

Custom action jobs with Lambda - https://docs.aws.amazon.com/codepipeline/latest/userguide/actions-invoke-lambda-function.html
- KEY IDEA #1: Hook up a Lambda function to do anything you want, but need more than basic permissions for its role - need CodePipeline access to write back the results PutJobSuccessResult, PutJobFailureResult
- KEY IDEA #2: You need to pass back a continuationToken to CodePipeline, this tells CodePipeline where it triggered in its execution, where to continue operations

Automated Integration API Testing https://aws.amazon.com/blogs/devops/automating-your-api-testing-with-aws-codebuild-aws-codepipeline-and-postman/

CodePipeline/CloudFormation actions
- Use CodePipeline to deploy CloudFormation
- But can also create CodePipeline using CloudFormation - READ THESE EXAMPLES https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-cloudformation.html
- Can mix and match - READ AND DEPLOY project - DevOps best practices https://github.com/aws-samples/codepipeline-nested-cfn
- Advantage: can reproduce as many times as you want. Note for example CodePipeline source references a particular branch - would have to create another one for each branch. Two options:
  - Clone pipeline with clone command
  - Use CloudFormation to create multiple (better!)
- As DevOps always have to think: how do I scale this

Security
- User IAM security - to be able to invoke the pipeline
- Pipeline has a service role (under Settings) 
  - NOTE that users don't need these permissions, they only need to be able to invoke the pipeline
- Manual approvals - Which IAM users or roles can approve
- Artifacts - S3 key or own KMS key

CodePipeline automations
- Already integrated with sources like CodeCommit or GitHub
- Notification rules
  - Detail
  - What are triggers for the notification
    - actions, stages, pipeline
	- started, succeeded, canceled, resumed, etc.
	- also for manual approval
- Custom Action provider
  - Invoke Lambda
  - Choose which artifact(s) to send
- Triggering pipeline from other things in AWS
  - CloudWatch/EventBridge event rule
  - Scheduled (e.g. set up a weekly build) or based on source/patterns
