# Secrets Manager
- Stores secrets
- How different from SSM Parameter Store with SecureString values
  - Credential rotation
  - Tight integration with RDS - so special secret types like "RDS database credential", "DocumentDB credential", etc
  - A little more expensive because you pay for storage per-secret plus usage (like Parameter Store)
  - Each secret stores multiple values (see below)
- Types
  - DB credential secrets 
    - just username and password
    - KEY IDEA but it also updates the DB to set the secrets (including if they are rotated) - so more secure, you don't have to do manually or build it
  - "Other types of secrets" - multiple key/value pairs
- Default encryption key or specific KMS key

### Automatic rotation
- generates new secrets and updates services to use them
  - Specify an interval and a Lambda function to generate a new credential or pull down from somewhere
  - AWS provides standard services or you can customize
  - Lambda called several times for different steps in rotation: create/set/test/finish
- Enable or disable; NOTE it says disable if you are using it. Does this mean ou can't rotate once you start using it???
  https://docs.aws.amazon.com/secretsmanager/latest/userguide/getting-started.html?icmpid=docs_asm_console#term_rotation