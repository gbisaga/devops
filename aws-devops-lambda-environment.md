Environment variables
- Basic configuration method in Lambda definition - specify the variables directly, key/value
  - Also can specify encryption in transit - specify a KMS key
    - Passed into Lambda function encrypted - use boto3 to decrypt the variable's value
    - The Lambda's role needs to have KMS policy (Decrypt operation - for specific KMS key or all)
- Better way to deal with secret values - Systems Manager (SSM)
  - Parameter store - externalize any commonly needed parameters
  - Can be SecureString to import secret values
- Even better for secure value is Secrets Manager (more later)

Lambda versioning - these are called "qualifiers" in the console - 2 kinds of qualifiers
- Versions
  - When you work on it, it's $LATEST - mutable
  - When happy, you can publish it as a version
  - Versions are immutable - increasing version numbers
  - Each version gets its own ARN
  - A version is code + configuration - and is always immutable
  - You could directly access the function versions
- Aliases
  - Like a pointer to any arbitrary Lambda version
  - Change it over time as you update code
  - Example: DEV alias (maybe $LATEST), then TEST alias to point to v2, PROD alias to v1
  - For Blue/green canary testing - specify weights on the alias - e.g. "PROD = 95% to v1, 5% to v2" for testing
- The ARN ends either with a number (for a version) or text (for an alias, which I suppose can't be just a number...)
- For an alias, log output shows which version is actually running
- Note that CodeDeploy for Lambda in canary or linear modes actually uses weighted aliases under the covers when doing Lambda deploys
  - e.g. LambdaLinear10PercentEvery2Minutes
  - How does this actually work? CloudWatch events?

SAM CLI
- template.yaml file - this is CloudFormation, but needs a transformation AWS::Serverless-2016-10-31
  - AWS::Serverless::Function - for a Lambda function
  - Events - triggering events
  - Can easily create an API Gateway resource
  - Outputs - API, Lambda ARN
- Automatically handles dependencies e.g. through requirements.txt file for python
- Include unit tests
- Sample includes sample events for local testing
- Can run functions locally
  - requires having docker installed to run functions
  - can even run a local API Gateway
- Package function `sam package` - generates a CloudFormation template and packages and uploads bundle to S3
- Deploy - may need to give capabilities e.g. create IAM role (normally asks in the UI of CloudFormation)

SAM also comes with CodeDeploy built-in to automatically do traffic shifting, blue/green deployment
- Automatic create new versions and aliases
- Gradually shift traffic
- Pre and post traffic test functions
- Rollback if CloudWatch alarms are triggered
