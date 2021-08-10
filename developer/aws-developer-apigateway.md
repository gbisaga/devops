### API Gateway - building a serverless API
- handles API versioning
- handles different environments (dev, test, prod...)
- Security (authentication / authorization)
- API keys, request throttling
- Swagger/Open API import to quickly define APIs
- Transform and validate requests/responses
- Generate SDK and API specs
- Cache API responses
- Integrations
  -- outside VPC, EC2, ALB, any AWS service, proxy HTTP(s)
  -- inside VPC - Lambda, EC2
- Best practices (from webinar)
  - Define stage variables and use them instead of hardcoding - for lambda, for HTTP endpoint (can't use for method though)
  - Control access to your APIs
    - Data plane - "execute-api:*" permission - who can invoke your API - for your customers
	- Control plane - "apigateway:*" permission - who can manage your API - for your developers
  - Environments, etc
    - With Lambda integration, use ALIAS instead of VERSION
	- Use API Gateway stages for ENVIRONMENTS not VERSIONS - easier to manage
	- Have different "stacks" for different environments. Best case is different accounts for different environments.
	
### Deployments and Stages (EXAM)
- Changes may not become effective immediately - make a "deployment"
- Deployed to any number and name of "stages" (dev, test, beta, etc.)
- each stage has own config parameters
- Stages can be rolled back
- Stage variables - like environment variables for API Gateway
- common Use case - Configure HTTP endpoints stages talk to (i.e. which to proxy to)
  - passed to Lambda thru mapping templates
- Common to use with lambda aliases (i.e. versions) - so DEV stage can call DEV alias (or whatever) in Lambda
- Canary deployments - can use in any stage, usually production - choose % of traffic the canary channel receives, e.g 5%
- Metrics and logs separate for better monitoring
- blue/green deployment with Lambda and API Gateway

### Mapping templates - modify request/responses - rename, modify body content, add headers, etc
- Use Velocity Template Language
- Hands-on example, changed JSON to XML

### Swagger/OpenAPI
- Import Swagger/OpenAPI 3.0 special
- Request/responses plus AWS extensions
- YAML or JSON
- Export from a stage

### Caching reduce number of calls to backend
- Default TTL 300 sec (0-3600)
- Defined at the stage level - can Override on specific methods
- Encrypt option
- Cache capacity 0.5GB - 237GB
- Client can request bypass ==> Cache-Control: max-age=0

### CloudWatch logs, metrics, XRay tracing
- Defined at stage level
- XRay - can trace requests all the way through the system

### API Gateway - CORS
- Make API gateway 
- Enable OPTIONS pre-flight - 3 headers Access-Control-Allow-*
- Creates a mock OPTIONS endpoint

### Usage plans
- API keys

### API Security - 3 aspects
- 1) IAM permissions - attach IAM policy, Gateway verifies IAM permissions. Good to provide access within your own infrastructure
  - "Sig V4" capability with IAM credentials in the headers. Watch for "SIG V4"
- 2) Lambda or "Custom" Authorization - uses Lambda to validate token in the header. 
  - Can cache the result
  - Helps for 3rd party authentication (OAuth, SAML, other)
  - Lambda returns IAM policy for the user
- 3) Cognito user pools
  - Cognito manages user lifecycle
  - No custom implementation needed
  - Cognito only helps with authentication/identification, not authorization. Backend code must ensure user is authorized to make the call.
  - Client calls Cognito to authenticate, gets token back. Then call REST API + Pass token
- Compare use cases
  - 1) IAM - good for user/roles in your AWS account; authentication + authorization; uses SIG V4
  - 2) Custom/Lambda authorizer - good for 3rd party tokens, flexible in what IAM policy returned; can handle auth+auth; pay per Lambda invocation (but htere is cache)]
  - 3) Cognito user poo; - manage own user pool
