CloudFormation CreationPolicy
- Create ASG with CloudFormation stacks
  - Define AWS::AutoScaling::AutoScalingGroup
  - AvailabilityZones option - can use Fn::GetAZs with your region reference AWS::Region
  - LaunchConfigurationName
  - MinSize/MaxSize/DesiredCapacity
  - KEY IDEA To ensure the ASG is created correctly, add a CreationPolicy - wait until 3 instances, up to 15 minutes (PT15M)
- Create AWS::AutoScaling::LaunchConfiguration in same file - ImageId, InstanceType, Fn::Base64 of user data 

CloudFormation UpdatePolicy
- CloudFormation doesn't always recognize there's an update in a template: "To update an AWS CloudFormation stack, you must submit template or parameter value changes to AWS CloudFormation. However, AWS CloudFormation won't recognize some template changes as an update, such as changes to a deletion policy, update policy, condition declaration, or output declaration. If you need to make such changes without making any other change, you can add or modify a metadata attribute for any of your resources."
- Ex: Change LaunchConfiguration to change image or size for example, update stack replacing template
  - Picks up changes to the launch configuration
  - HOWEVER, didn't relaunch the instances - ASG was updated, but not the instances
  - So, if you changed the userdata it wouldn't take effect
- KEY IDEA By default changes ASG but not the instances. To fix this: specify UpdatePolicy 3 options https://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-group-rolling-updates/
  - AutoScalingReplacingUpdate
      WillReplace: 'true' <- ASG immutable - create a whole new ASG, terminate the old 
  - AutoScalingRollingUpdate:  # Keep same ASG, Do a Keep at least 1, do 2 at a time
      MinInstancesInService: 1
	  MaxBatchSize: '2'
	  PauseTime: PT1M
	  WaitOnResourceSignals: 'true'
  - AutoScalingScheduledAction:             ???

CodeDeploy with ASG
- Nice integration - CodeDeploy deploys and keeps all instances up to date
- CodeDeploy deployment group
  - EC2/on-prem type
  - Environment configuration, but tell it to use an ASG - deploys to all instances in the ASG
  - If ASG exists with instances, when you start a new deployment with CodeDeploy, it will deploy to the ASG instance(s)
  - Then newly deployed ASG instances will also get the same deploy
- KEY IDEA If you use CodeDeploy blue/green and an ASG, you MUST hook it the deployment group with a load balancer (otherwise it wouldn't be able to shift traffic). It will provision a new ASG, connect the LB to it, and terminate the old ASG. CodeDeploy is really a smart service to manage deployments to ASG/LB!
  - Step 1. Provision replacement instances
  - Step 2. Install application
  - Step 3. Reroute traffic to replacement instances
  - Step 4. Terminate original instances
- But note: what if you do a deployment but while that is happening, there's an autoscaling event and a new instance is created?
  - It gets the old version, because that is what was current at the time, so you'll have a mix of versions
  - KEY IDEA read https://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-aws-auto-scaling.html
  - Two options:
    - Do a second deploy
	- Use ASG suspended processes - suspend the Launch process

ASG deployment strategies with CodeDeploy
- In place (1 LB, 1 ASG) - everything same, replaced in-place
- Rolling (1 LB, 1 ASG) - create new instances, then delete
- Replace (1 LB, 2 ASG)- create new ASG with new instances but the same ALB; so at some points ALB points to both ASGs
- Blue/Green (2 LB, two ASG, R53 change)
