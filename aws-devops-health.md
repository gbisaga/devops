AWS Health - several ways of using
- First is global health dashboard 
  - Service Health Dashboard - health of AWS itself and its services
  - All services, all regions - problem is, it's too big
- Personal health dashboard
  - Event log for things that apply to you
  - Click on bell to see
  - Things like operational issues with service (lambda/EC2/ELB/etc), upgrades, VPN redundancy loss, security patching
- Or set up SNS notification with CloudWatch events
  - Health > Specific Health Events 
  - github AWS Health AWS_RISK_CREDENTIALS_EXPOSED
  - Can scan github looking for public IAM keys -> CloudWatch event -> Step functions that invoke lambdas to:
    1. Remove role from IAM
	- Get audit from CloudTrail
	- Output notifications to SNS
- KEY IDEA Use a STEP FUNCTION rather than just a Lambda, because it is much easier to debug problems or track progress
