AWS Parameter Store - like Key Vault
- Secure storage for config/secrets 
- Optional Seamless Encryption using KMS
  - Can store encrypted or plaintext parameters
- Serverless, scalable, durable, easy SDK, free
- Version tracking
- Configuration management using path and IAM
- Notification of changes with CloudWatch events
- Integration with CloudFormation -> feed parameter store secrets to CloudFormation template
- Parameter store hierarchy is the "path" - like a Unix filesystem tree - organize however you want (around apps, departments, etc.)
  - Can differentiate by environment using part of the path
- APIs: GetParameters or GetParametersByPath
- get parameters from CLI with or without encryption (with requires access to the KMS key)
  aws ssm get-parameters --names /my-app/dev/db-url /my-app/dev/db-password --profile=powerschoolapollo --with-decryption
  aws ssm get-parameters-by-path --path /my-app/dev/ --profile=powerschoolapollo
  also --recursive
