ECS - service autoscaling
- CPU and RAM tracked in CW at ECS Service Level
- Target Tracking: average
- Step scaling: based on alarms
- Scheduled scaling: predictable changes
- ECS service scaling (task level) <> EC2 autoscaling (instance level)
- Fargate much easier (serverless - no provisioning)

ASG scaling
- Cluster capacity provider
- Determine infrastructure task runs on
- For ECS on EC2, associate with an ASG
  - Will launch new instances if needed, even if the ASG wouldn't do it on its own (e.g. CPU not high enough)
