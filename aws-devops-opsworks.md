### OpsWorks - using Puppet and Chef to provision and configure infrastructure
- "AWS OpsWorks is a configuration management service that helps you build and operate highly dynamic applications, and propagate changes instantly"
- 3 options
  - Chef Automate - creates and uses a standalone server, not in exam
  - Puppet Enterprise - not in exam
  - Stacks - uses Chef cookbooks to deploy application
- Sounds a lot like other tools - https://tutorialsdojo.com/elastic-beanstalk-vs-cloudformation-vs-opsworks-vs-codedeploy/
  - note that SSM can also run chef recipes
  
### Stacks
- Infrastructure, config, and application 
- Chef uses cookbooks to provision and config the application
- Specify custom Chef cookbook pulled from a repo - S3, HTTP, or git
- Has a subset of VM configs such as IAM role, root device type, API endpoint region, etc.
- Contains multiple layers aka blueprints for set a EC2 instances
  - Layer has a bunch of parameters like shutdown timeout, auto-healing 
  - Layers have a bunch of Chef recipes you can apply to different lifecycle events (setup, config, deploy, undeploy, shutdown)
  - Layers can also be ECS clusters or RDS instances (only registered - not managed by OpsWorks)
- Stack also manages multiple instances
  - Each instance has typical EC2 settings
  - Need to start the instances 
  - Have to stop instances to edit their settings
  - Instances provisioned and managed by OpsWorks
  - Special kinds of instances - create many, managed by OpsWorks
    - Default is 24/7 type (always on)
    - Time-based - OpsWorks starts and stops instances based on a schedule, simplified cron-type UI
	- Load-based - starts new instances based on layer metrics or up and down on CloudWatch alarms
  - Note that this is NOT autoscaling 
    - you have to predefine the specific instances you want. 
    - OpsWorks just starts and stops them.
- Apps specifies where an application comes from (e.g. github)
- Deployments - can run specific Chef cookbooks remotely on one or all instances
- Monitoring the layers
- Can color-code stacks to quickly show production vs test, etc.

### KEY IDEA Summary: Stack has:
- many layers
- 3 types of instances (24/7, time-based, load-based), each one has events/logs
- lifecycle events with Chef cookbooks as hooks
- one or many apps deployed onto instances
- monitoring
- run remote commands

### KEY IDEA: stack layers have 5 lifecycle events with associated set of recipes run after the event's built-in recipes
- Note: all these are instance specific EXCEPT config, which runs everywhere when something happens on any instance
- setup: after an instance finishes booting
- config: KEY IDEA runs on ALL stack's instances in situations for any instance: up or down instance, add or remote an elastic IP or ELB
- deploy: deploying application to instances - setup includes deploy
- undeploy: delete an app or undeploy command
- shutdown: shutting down an instance, but before the EC2 is terminated - this also runs the config hook
