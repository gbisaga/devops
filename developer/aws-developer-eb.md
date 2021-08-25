# Elastic Beanstalk Deployment - EXAM questions
- Deployment modes
  - Single instance - one EIP, ASG, SG - great for dev
  - High availability with or without LB - ASG, across multiple AZs, one or more - great for prod
    - with LB for web workloads
    - without LB, only with ASG for worker processes (handling messages from SQS, cron.yml)
- Deployment options for Updates
  - All at once - deploy all in one go - fastest, but downtime
  - Rolling - update a few instances at a time (bucket), then next bucket when the first is healthy. Application will keep running, but at lower capacity. Can set bucket size (# of instances stopped at a time). No additional cost since no additional instances created.
  - Rolling with additional batches - spins up new instances to move the batch, so the old application is still available Application keeps running at same capacity, also set the bucket size. Additional cost for temporarily left-around older instances, but usually not very much.
  - Immutable - spins new instances in new ASG & swaps all instances when everything is healthy. Also zero downtime. But others were updating previous instances, this is a new one. More expensive, but quick and easy rollbacks.
- Uses CloudFormation under the hood

### Blue/green - not a direct feature. Zero downtime and release facility with more testing.
- Create a new "stage" environment (green), valid independently and roll back if issues
- Set up Route 53 to redirect a bit of traffic to the stage environment
- Using Beanstalk, swap URLs when done with the environment test

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

### KEY IDEA for exam
- Additional EB CLI makes working with Beanstalk easier - eb create (deploy application version to env), eb status, eb health, eb events, eb deploy, etc. Good for pipelines.
- Worker associates with SQS queue, pulls from it and calls localhost:80, where you write handler to process messages
- Speed optimization: Instead of using package.json/requirements.txt etc and having EC2 machines resolve dependencies, do it yourself first and upload the whole zip file. Faster deployment.
- Beanstalk with HTTPS - load SSL cert onto LB from console or .ebextensions, files end with .config. SSL certificate can be provisioned from ACM. 
  - Must also configure SG rule for 443. (Q: Doesn't EB do this by itself?)
- Redirect HTTP to HTTPS - or configure ALB with a rule. But make sure to still allow HTTP from LB for health.
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/configuring-https-httpredirect.html
- Lifecycle policy - can store at most 1000 application versions. Need to create a lifecycle policy - two kinds, based on time or space. Versions in use won't be deleted. Option not to delete underlying source bundle in S3.
- Web server vs worker environment. Long running tasks - better in worker environment. 
  - Common design pattern: Decouple application into web and worker tier. Example: processing a video, generating a big zip file, etc. 
  - Can define periodic tasks with cron.yaml extension file. Use a work queue (SQS) to send work to the worker tier in a new environment.
- Manage RDS with Elastic Beanstalk. Can create RDS database - great for dev or test, but not for prod. Question: how do you decouple RDS coupled in EB to standalone if you need to?
  - Take RDS DB snapshot
  - Enable deletion protection in the RDS
  - Create new environment without an RDS, pointing to existing RDS
  - Blue/green deployment to swap
  - Terminate old environment - RDS won't get deleted because of deletion protection
  - Delete CloudFormation stack (will be in DELETE_FAILED state)
