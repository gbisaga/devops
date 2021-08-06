Lambda - serverless
- Don't have to manage servers - we just deploy code. FaaS - Function as a Service
- Serverless now includes anything that's managed
- AWS Lambda and Step functions - DynamoDB, Cognito, API Gateway, S3, SNS/SQS/Kinesis, Aurora serverless. Now Fargate and SageMaker
- vs EC2 
  -- EC2 - SERVERS, limited by RAM/CPU, continuously rnning, scaling means add/remove servers = slower
  -- Lambda - FUNCTIONS, time-limited, run on-demand; automated, instantaneous scaling
- easy pricing, only pay per request and compute time; more RAM = more CPU and network
- integrated with whole AWS stack, so it can be used as glue with many other services.
  - built-in integrates with many services: API, Kinesis, DynamoDB, S3, IoT, CloudWatch events, CloudWatchlogs, SNS, Cognito, SQS
  - also custom "partner" event sources via EventBridge
- all major dev platforms - node, pythong, jvm, c#, golang, C#/powershell
- example: thumbnail creation; S3 triggers> Lambda > pushes new thumbnail to S3, metadata to DynamoDB
- another example: serverless cron job --> CloudWatch triggers cron events > Lambda to perform some task
- pricing: pay per call (first 1M free), plus duration (400GB-seconds of compute time/month free)
  -> it's usually VERY CHEAP to run AWS Lambda

Configuration
- Default timeout 3 sec, max 15 minutes
- Environment variables
- Allocated memory - 128M to 3GB
- Can deploy in a VPC, assign security groups - if in a VPC, function is a little slower to start
- Lambda always needs a role - AWSLambdaBasicExecutionRoleXXXX - always needs at least limited write to CloudWatch logs for the logs
- Environment variables - options to encrypt at rest and in transit
- Can run custom Lambda runtime environment - AWS defines HTTP API to manage.
 https://aws.amazon.com/about-aws/whats-new/2018/11/aws-lambda-now-supports-custom-runtimes-and-layers/
- Layers - for common code libraries. Upload a ZIP to a layer then you can use it in multiple functions. Easier management since you manage them from the layer, not all the functions that use the layer. See above link.

Concurrency and throttling
- default 1000 executions (can increase with a ticket)
- can set a reserved concurrency at function level (reserve capacity)
- each invocation over concurrency limit triggers a "throttle" - throttle behavior
  - if synchronously (e.g. from console) - return ThrottleError - call is responsible to retry
  - if async - automatically retries 2x, then unprocessed events go to DLQ if you set it up; DLQ can be SNS topic or SQS queue to process later
  Ex: S3 is async trigger - so if problem with the image, executes 2x (can config to 0-2, and max age default 6h) then can go to the DLQ
  --> Makes it easy to debug code in production
  --> Make sure Lambda has correct IAM execution role, or it can't write to DLQ

Logging, monitoring, and tracing
- Lambda logs tsored in CloudWatch logs
- Lambda metrics displayed in CloudWatch metrics
- Must have execution role authorization writes to CloudWatch
- Easy to enable with XRay - normally have XRay demon, but run automatically
  --> Ensure Lambda has corrent IAM role

Lambda limits for EXAM
- 128-3008MB in 64MB increments
- Max execution time: 15 minutes, but exam might say 5...
- Disk capacity in "function container" /tmp directory - 512 MB
- Concurrency limit: 1000, can increase with ticket
- Deployment - compressed ZIP file must be < 50MB max; size of uncompressed must be < 250MB
  -> Could use /tmp directory to load other files at startup to get a little more space
- Size of environment variables - up to 4KB, can't store whole files

Lambda versions
- $LATEST - mutable - when ready to publish it, create a version. Versions are immutable, a snapshot.
- Each version has its own ARN and can be used at any time.
- Version = code + configuration (env vars, timeouts, etc.)
- Aliases 
  -- Pointers to a particular version. dev, test, prod alias to different Lambda versions. 
  -- External users often use aliase, not $LATEST or specific versions.
  -- Blue/green deployment by assigning weights - e.g. 90% points to V1, 10% points to V2
  -- Aliases have their own ARNs also.

External dependencies
- Only way is to package in same zip file - npm and node_modules, pip --target option, java .jar files, etc.
- Upload zip file straight into Lambda if < 50MB, else S3 first
- Native libraries work - just have to compile for Amazon linux (shows underlying Lambda server environment!)
- Automatically includes aws-sdk library

Lambda with CloudFormation
- Store Lambda zip in S3
- Reference S3 ZIP location in CloudFormation

Lambda /tmp space - for disk space or big files to download/install
- Remains when execution context is frozen (between invocations) - used for multiple invocations
- For permanent persistence, use S3

Lambda best practices
- Perform heavy-duty work outside function handler - Connect to DB, Initialize AWS SDK, pull in dependencies or datasets
- Use environment variables for DB connection string, S3 bucket - not in code. Passwords, sensitive values, encrypted with KMS
- Minimize deployment package size to bare necessaties
- Avoid recursive code - never have a Lambda function call itself
- Don't put Lambda in a VPC unless you have to - take longer to initialize

Lambda@Edge
- Say you deployed a CDN using CloudFront - want o run a global AWS Lambda alongside, or implement request filtering
- Lambda@Edge - Lambda not in a region, it's deployed in every region
- Four steps to a CloudFront request: (1) Viewer request (user->CloudFront) (2) Origin request (CloudFront to origin), (3) Origin response (origin to CloudFront), View reponse (CloudFront to user).
- Thru Lambda@Edge, can insert Lambda at any of these points ... so the Lambda could actually be the "origin"
- Use cases -- Website security, bot mitigation, image transformation

SAM - Serverless Application Model - framework to develop and deploy
- Simple SAM YAML code -> generates complex CloudFormation YAML code
- Supports anything that CloudFormation supports
- Only two commands to deploy to AWS
- Can use CodeDeploy
- SAM can help you run Lambda, API Gateway, DynamoDB locally - LOCAL TESTING!
- Header: Transform: 'AWS::Serverless-2016-10-31'
- Write code with 3 basic helper functions
  - AWS::Serverless::Function - Lambda
  - AWS::Serverless::Api - API Gateway
  - AWS::Serverless::SimpleTable - DynamoDB
- SAM Policy templates - we included this when we wanted to access DynamoDB for CRUD operations
  - List of templates to apply permissions - make it easy to apply permissions https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html
  - S3ReadPolicy, SQSPollerPolicy, DynamoDBCrudPolicy
  - Policies: 
     - SQSPollerPolicy:
		QueueName:
		  XXXX
