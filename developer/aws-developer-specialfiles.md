Special PaaS URLs:
- http://ENVIRONMENT_NAME.EB_ENV_SHORT_NAME.us-west-2.elasticbeanstalk.com/ or http://UNIQUE_ENVIRONMENT_NAME.us-west-2.elasticbeanstalk.com/
- ACCOUNTID.dkr.ecr.us-west-2.amazonaws.com
- http://s3-us-west-1.amazonaws.com/bucketname/objectname
- kinesis.us-east-1.amazonaws.com
- LONG_UNIQUE_IDENTIFIER.us-east-1.elb.amazonaws.com
- https://REST_API_ID.execute-api.us-east-1.amazonaws.com/STAGE_NAME/RESOURCE_NAME

Special files:
- .ebextensions/*.config - extension config files
- buildspec.yml - CodeBuild config
- appspec.yml - CodeDeploy config
- cron.yaml - cron in EB
- Dockerrun.aws.json - ECS config, also used in EB docker deploys
- bootstrap - custom Lambda runtime
