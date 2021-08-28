Integration with CloudWatch events: https://docs.aws.amazon.com/codedeploy/latest/userguide/monitoring-cloudwatch-events.html

Send to lambda, kinesis, SQS, build-in targets (cloudwatch alarm actions), SNS. Some use cases:
- Send to slack channel: Create Rule > CodeDeploy > State change > State change notification > Specific States > FAILURE;
  target = Lambda function, has sample event JSON
- Push data to a kinesis stream to show changes on a dashboard: Create Rule > CodeDeploy > State change > Any detail type, any application; 
  target = kinesis stream
- Automatically start/stop/terminate instances on success or failure: Create Rule > CodeDeploy > State change > Instance state change > Specific States > READY
  target = EC2 RebootInstances API call

For state changes, can be Deployment or Instance - same as triggers

CodeDeploy logs - install CodeDeploy agent and CloudWatch log agent - no automatic connection

NOTE that CodeDeploy is not a target for CloudWatch events/EventBridge

Notifications and Triggers - like CodeCommit
- Triggers to SNS only (no lambda unlike CodeCommit)
  - Deployment and Instance triggers - same as CloudWatch state change types
- As with CodeCommit, CloudWatch events/EventBridge are MUCH more flexible (many target types, can specify conditions, multiple targets for an event)

Think about the kinds of DevOps integrations you might want to do.
Read: https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/introduction-devops-aws.pdf

-------

Integration with CloudWatch alarms (not events!): thousands of types of metrics, per-instance or aggregated.
- Can use to stop a Deployment
- Rollback Deployment (if configured)

