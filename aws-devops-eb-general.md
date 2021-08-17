EB is just a tool to build and manage CloudFormation templates
- KEY IDEA: Two "tiers" - WebServer and Worker

EB CLI
- eb init --profile XXX
- eb create dev-env <- zip up contents to S3, creates env with SG, EC2s, ASG, LB, CloudWatch alarms, etc.; lots of output, same console or CLI
- eb open <- open browser to view application
- Note CLASSIC LB, not ALB
- Config file: .elasticbeanstalk/config.yml

Most important CLI commands
- eb status
- eb health (eb health --refresh to update every 10 second, equivalent to "health" console page)
- eb logs <-- logs for all the parts, same as "logs" console, but more logs
- eb deploy <-- redeploy app; this redeploys the same previous environment; you could put it in CodeCommit and run "eb deploy" using CodeBuild
- eb terminate <-- to delete whole environment

Reproduce an EB environment - back it up as code
- Could go to CloudFormation, get the template
- But directly in EB use "saved configuration" - more native to EB
  - eb config save dev-env --cfg initial-configuration
  - creates locally in .elasticbeanstalk\saved_configs\initial-configuration.cfg.yml
  - OptionSettings section - only the non-default settings - lots of default settings show up in the console
- eb setenv ENABLE_COOL_NEW_FEATURE=true <-- immediately updates the environment in AWS
- KEY IDEA: Saved configs are also good for disaster recovery

Change a configuration
  - Option 1) could do it from the console
  - Option 2) better to modify the saved config file and redeploy it
    - eb config put prod <-- updates saved config in AWS - assumes saved config file named prod.cfg.yml
    - eb config dev-env --cfg prod <-- change config according to saved config in AWS, NOT THE FILE
  - Option 3) Use .ebextension/*.config files - very flexible
  - Precedence: (1) options applied directly in environment (2) saved configs (3) .ebextension files (4) default values

Miscellaneous good to know info
- Environment types - Low Cost (free tier, single server, good for dev) and High Availability (ASG+LB, good for prod)
- Application versions and policies
  - Able to easily roll back to a previous version
  - Current application version - limit in EB, only can have 1000
  - Application version lifecycle policy
  - Keep only X versions, only keep for Y days, keep/delete source bundles (can roll back)
  - Never delete a version deployed to an environment
- Clone command - create a new environment (can also clone via CLI)
- Terminate environment (`eb terminate`)
- Rebuild environment - delete and recreate everything
  - Note this includes all resources e.g. databases, SNS topics
- Managed updates - apply patches to your environment on Weekly basis
  - Enable in configuration tab
  - Weekly update window, update level (e.g. minor and patch)

Worker environments
- Two major use cases - a single worker can do both
  - Long running workload (e.g encoding a video) - example program reads SQS queue to pick up work
    - The SQS is created automatically, or can use an external queue (so it's not deleted)
  - Perform scheduled work using a cron
    - KEY IDEA: cron.yml file

Can use docker to create EB application
- KEY IDEA: can use any environment standard you want, as long as you have a docker image for it. Not just the specific environment options they give you. (COBOL?)
- KEY IDEA: Dockerrun.aws.json is EB-specific docker configuration, used for multidocker 
- Two options
  - Docker
  - Multi-container docker - creates an ECS cluster
