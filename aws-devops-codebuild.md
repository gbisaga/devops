### Fully managed build service (like jenkins) 
- always serverless, pay for usage to complete build
- Docker under the hood - reproducible
- Can provide own docker images
- Secure
- Get code from anywhere
- Build buildspec.yml
- Notifications 
  - Logs to CW logs or S3
  - Metrics to monitor CB statistics
  - CW Events for failed builds, trigger notifications
  - CW Alarms to notify 
  - CW Events/Lambda for glue
  - SNS notifications with CB trigger

### Create Build project
- Specify source - can be CodeCommit, S3, github, etc.
- Specify a branch, tag, or commit 
  - So multiple branches requires multiple CodeBuild projects
  - This obviously encourages simpler branching!
- Specify "additional configuration" e.g. timeouts up to 8 hours
- So better suited for something like performance test, can be much longer than Lambda
- Can specify a VPC if resources are in a VPC
- Can specify compute resources - more = pay more
- Finally most important part - specify a build spec file or inline commands if very simple. For us, default name (buildspec.yml), and it's in the repo
- Then specify artifact and logs locations - need to make logs go to CW or S3 else lose logs
- Each CodeBuild project can have a build badge for the README.md - Create > Copy Build Badge URL
- Can run scheduled trigger (e.g. daily)

CodeBuild types of Sources
- CodeCommit
  - No direct build source trigger - need to use CloudWatch event/EventBridge
- S3 - add a bucket and object key
  - No direct build source trigger - need to use CloudWatch event/EventBridge (need CloudTrail integration also)
- GitHub or GitHub Enterprise - either public repo or connect to your GitHub account
  - Can create a webhook on GitHub to trigger a CodeBuild job
  - Can report status - so you can have GitHub PR's require successful tests
- BitBucket - similar but no web hook

Best way to test code? Probably CodeBuild, serverless, very little setup required

buildspec.yml - https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html
- Specify an "env" section with variables, parameter store or secrets manager (e.g. login password)
  -> What's the diff? Both can be encrypted, SM is always; PS is free, SM $0.40/secret + access fee; SM allows autorotation, autogeneration https://acloudguru.com/blog/engineering/an-inside-look-at-aws-secrets-manager-vs-parameter-store
- Specify different phases -> These are the only ones install, pre_build (e.g. login to ECR), build, post_build
- Install can specify the runtime version (e.g. node 10)
- Each phase can have `commands` block, plus a `finally` block to cleanup
- Specify artifacts to be uploaded (else lost in the docker container)
- Purpose of pre-build - FAIL EARLY - so for example (docker example) don't wait until later when you might waste the time to do the build, only to fail on the ECR login.
- Can create reports - `reports/SurefireReports:` element
  - Shows overall status, results of each test
  - Reports expire in 30 days (can copy artifact to S3)
  - Can show trends - make sure not getting worse in testing

------
Samples for CodeBuild = https://docs.aws.amazon.com/codebuild/latest/userguide/use-case-based-samples.html
Ex: Docker Sample for CodeBuild - build docker image, and upload to ECR
-> https://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html <- this is a useful sample, since we'll be tested on it

Build Environment 
- Can specify your own docker image - can speed up the builds, including libraries you need, etc
- Environment variables - https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html
  - two types - PLAINTEXT or PARAMETER_STORE (also Secrets Manager now)
  - printenv - prints the vars
  - can either put in buildspec.yml (env/variables section) or specify as overrides from the console/CLI
  - They are visible - so don't put sensitive data in these! DB URL is probably fine as an Environment variable.
  - But DB password is bad - anybody can see it, it can be logged, etc.
  - So we can put in Systems Manager parameter store - type = SecureString (encrypted with KMS). Then the parameter name is the value you specify for the environment variable.
    - But note: the IAM you have probably doesn't include parameter store. So you need to attach an IAM policy to access parameter store, at least read-only.
    - Also note: This is just added as a unix env var... so you can still print them out using printenv, etc.
- Filesystems
  - Add EFS mounts to CodeBuild job's docker container

Batch build
- Trigger multiple builds

Security aspects
- User IAM
- Artifact security
- Service role - not assumed by a user, assumed by the CodeBuild service
  - Permissions CodeBuild requires during the build
  - Separate from user roles

Artifacts
- Maybe none (e.g. testonly, or docker push to ECR)
- Can specify file pattern and destination, currently only S3
- Be default - uses encryption in the build definition, but can turn off and/or specify default encryption for the bucket
- Need IAM role - "Allow AWS CodeBuild to modify this service role so it can be used with this build project" - updates "based policy" for the role automatically
- Note: Artifacts bucket needs to be in the same region!

Artifact security
- Source - corrupted or malicious source results in corrupted builds
  - Authenticated git repos (GitHub, BitBucket) - most secure
  - S3 (also secure)
  - Public git repos - not very secure, anybody could change
- Network
  - By default not in a VPC - easy setup
  - Can run in a VPC - but harder because all artifacts also need to be in the VPC
- Runtime - CodeBuild jobs
  - By default: AWS docker images - always runs latest - could potentially be a zero-day vulnerability
  - Your own docker images - you can control release cycle, patches, etc. - a lot more effort
  - NOTE AWS guarantees the environment completed deleted after each build
- Storage - where artifacts end up - encrypted by default (can disable encryption)
  - S3 - encrypted at rest - KMS encryption or S3 encryption
  - ECR - great for docker images - private registry
  - Store artifact to anything with an API via bash shell script - works for custom use cases, but it's the hardest option; have to be careful with security
- Access - see or modify pipeline settings
  - IAM policies - protect user access to CodeBuild, S3, ECR
  - S3 bucket policies
  - KMS keys - keys have their own policies

