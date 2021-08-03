Advanced ASG topics and integrations

Suspended processes
- KEY IDEA cause autoscaling processes to be suspended and resumed https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html (important in exam)
- E.g. suspend "Launch" process - will not launch any new instances, even if you manually change the desired capacity
- Use cases - Troubleshooting, allowing for temporarily skipping health checks (if you know it's healthy but it doesn't show that way)
- Processes you can suspend
  - Launch or Terminate - don't change number of instances
  - HealthCheck - don't check health the normal way (EC2 or ELB); can still set health manually using the CLI.
    - NOTE: if you're using a LB, the LB still will send traffic or not based on health checks; but the instance won't be deleted because the HealthCheck process is suspended
  - ReplaceUnhealthy - if instance is unhealthy, ASG won't replace it
    - Usually a better choice than HealthCheck - the health status still shows up, but instance won't be terminated and replaced
  - AZRebalance - if too many instances in one AZ, terminate and recreate to balance the instances
  - AlarmNotification - if scaling policies linked to an alarm, alarm won't trip the policy
  - ScheduledActions 
  - AddToLoadBalancer - don't automatically add new instances to the Target Group
    - Note: if you resume the AddToLoadBalancer process, instances created while it was suspended NOT automatically added to the TG!
	- Need to manually register the instances to the TG

KEY IDEA Troubleshooting instances in an ASG without impacting operations
- Can do it live, perhaps with suspending some processes
- Better option: DETACH the instance -> creates a new instance in the ASG -> replaces this instance in the TG -> but leaves the instance running
- Best option: SET TO STANDBY -> removes this instance from the TG but leaves it in place for troubleshooting (option to add a new instance to TG) -> when done set back to InService; can also use for handling long-running actions e.g. after reading a request from SQS
- Set Scale-in protection to certain instances - these instances will never be terminated even if desired capacity is reduced

KEY IDEA Autoscaling lifecycle hooks
- Gives control over instance creation and termination in an ASG
- Normally ASG instances have states Pending -> InService -> Terminating -> Terminated (these show at instance lifecycle status)
- Extra Pending and Terminated states - :Wait and :Proceed
- Use for hooks
  - Pending:Wait - if it takes a long time to install an application
  - Terminating:Wait - take a snapshot of EBS volume, dump logs to S3, etc.
- How to invoke lifecycle hooks? 3 ways, 2 legacy
  - SNS, SQS (legacy) - problem is you need something to catch these
  - CloudWatch events invoking a Lambda
- To create : ASG > Lifecycle Hooks
  - Hook either on Pending or Terminate
  - Heartbeat timeout in seconds, default action (PROCEED or ABANDON)
  - Can have a notifification target ARN and role (for SQS or SNS)
  - Better choice CloudWatch Event 
    - Service: Autoscaling, EventType: Launch and Terminate
	- Optional: specific events and specific ASG launch templates
	- In event detail JSON, have LifecycleActionToken -> a UUID used to continue processing
    - Target: Invoke a Lambda function to do whatever you need
	- KEY IDEA When Lambda is done, needs to call complete-lifecycle-action API call with the LifecycleActionToken or instance id to PROCESS or ABANDON in lifecycle - can 
  - Either way, if create a Pending lifecycle event, instance goes from Pending -> Pending:Wait -> then into InService when you call complete-lifecycle-action
- KEY IDEA Question: can you call a script in the EC2 instance? Yes, calling SSM EC2 Run Command with a SSM document (Must have SSM agent installed)
  - Lifecycle hook -> CloudWatch event -> Lambda -> SSM Run Command
  - TODO https://github.com/aws-samples/aws-lambda-lifecycle-hooks-function

Termination Policies
- Question: if there's an autoscaling action, which instance(s) are terminated first?
- Several policy options (could specify more than one?)
  - KEY IDEA Default 
    - find which AZ(s) has the most instances and at least one instance not protected from scale-in; 
	- then find oldest launch template or configuration; 
	- then closest to next billing hour (i.e. whose used the most of its billing period)
	- then pick one at random
  - OldestInstance - good for upgrading instances to a new EC2 instance type
  - NewestInstance - good for testing a new launch configuration but don't want to keep in production
  - OldestLaunchConfiguration - phasing out old configurations
  - ClosestToNextInstanceHour - not really useful since EC2 now billed by the seconds
  - OldestLaunchTemplate
  - AllocationStrategy - spot vs on-demand mix

SQS Integration
- Common when you're setting up workers doing backend processing (image processing) with a SQS queue feeding it
- Want to scale your ASG based on something more than number of messages in the queue 
  - Need a custom CloudWatch metric for the #messages/#instances
  - DIVIDED by the number of instances
  - more complicated, based on backlog, etc.
- KEY IDEA Want to protect the instance from termination while it's processing a message (OK but requires message to be handled again) - of if instance has a special role like Hadoop cluster master
  - Protect from scale-in at beginning of processing, then remove protection at the end

Monitoring 
- CloudWatch metrics
  - ASG related - min/max/desired, number of instances in each state, etc
  - EC2 group metrics - average CPU utilization, disk read/write, etc.
- Notification to SNS topic (launch, terminate, fail to launch or terminate) or CloudWatch events - as usual CloudWatch more flexible
- KEY IDEA No automatic ASG integration with CloudWatch logs - so use CloudWatch agent on EC2's WITH proper IAM role permission
