### Create an API - either REST or WebSocket
- REST - import swagger or OpenAPI or create your own
- Endpoints
  - Regional - public, within whatever region
  - Edge optimized - deployed to all CloudFront edge endpoints
  - Private - deployed within the VPC

- Create a method - Integration type
  - Lambda - integrate with a Lambda function
    - Create simple Lambda function - give name
    - Could have a custom API gateway timeout - default is 29 sec
    - API Gateway asks for permission to access the lambda function
    - Also LAMBDA_PROXY
      - Lambda can specify a Lambda alias or version - so either a specific version or using an alias you can do canary or A/B testing.
      - Recommended - else need to write VTL. More unit testable if everything in code.
  - HTTP - Proxy the request to your own service elsewhere
  - Mock - just mock it up
  - AWS Service - 
  - VPC link
- KEY IDEA: API is not usable until you deploy it to a stage 
  - As many stages as you want, any names.
  - Either dev/stage/prod, or you can version your APIs /v1 /v2
  - They can also be rolled back.
- Can create API Keys
- Can create CloudWatch log integration
- Can create authorizers - Cognito or Lambda
  - Read about cognito https://k21academy.com/amazon-web-services/aws-solutions-architect/amazon-cognito/

### Stage variables - like environment variables for the API Gateway stages
- Use lots of places
  - Lambda function ARN
  - HTTP endpoint
  - Parameter mapping templates
  - Passed in "context" object in Lambda
- Example: Set PROD stage variable to a Lambda PROD alias, then we can canary it v1 95/v2 5%; while the TEST stage variable points to a TEST stage alias, which is v2 100%

### API Gateway also has its own canary capability
- Go to Stage Editor for PROD, create a canary
- For that stage now there are two "paths" - the main one and the canary one
- Then you "promote" the canary path to the main path for the stage
- Notice that each stage can have its own canary

### Throttling - KEY IDEA there are three kinds of throttling with API Gateway
- 1) By default, 10k/sec avoid DDoS attacks
- 2) Can also define usage plans for each API stage
  - Each method can have its own throttling
  - API keys can link to usage plan
- 3) Lambdas also have throttling

### API Gateway can front-end Step Functions
- Could go thru a Lambda first
- But can specify integration type = AWS Service
- Can specify virtually ANY AWS service
- Need to give API Gateway an IAM role with permission to invoke service
