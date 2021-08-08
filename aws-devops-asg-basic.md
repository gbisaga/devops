ASG
- Create launch configuration - similar to creating an EC2 instance
  - ASG uses it
  - Many times need user data so new instances in the ASG can be set up right
- Create ASG specifies launch configuration, scaling policy
  - Create tags for ASG but "tag new instances" to pass them on to instances it creates
  - Other tags automatically added aws:asg:groupName
- Can also create ASG from Launch Template
  - New way to do it
  - KEY IDEA Similar to Launch Configurations, but more flexible: multiple instance types, combination of instance types of purchase options (on-demand and spot); also used for other services
- ASGs have a capacity with min/max/desired
  - Min and max are min/max instances possible
  - Desired is number of instances you want; scaling policies bump this number up or down and the ASG responds by creating/deleting instances to match
  - Note they are immutable - to change, create a copy and use that (launch templates are versioned)

Launch templates
- Revolutionary way to define instances
- Like a launch configuration, except can be used with several services (EC2 instance, ASG, Spot Fleet) - in future probably many more
- KEY IDEA Governance - best practices for EC2 across organization with various services
- Templates are versioned 
  - mark one as default version to be used with new instances
  - tracks with instances you create with them (instance X built from template Y)
- Can inherit from another template
- Include only the parameters you want - AMI, instance type, key pair, security group, etc.
- Define instance tags - can be passed on to EC2 and disks if you want
- User data, etc.
- Can specify % combination of on-demand and spot instances (not in launch configurations)

Scheduled actions
- Maybe you know patterns of when users will show up or not - scale in or out
- Change one or more of the scaling values (min/min/desired) scheduled in advance
- Choose "run once", "periodic" (at time of day) or "cron"

Scaling policies - now called Dynamic Scaling Policy
- Unlike scheduled actions, we don't specify min/min/desired - options
  - Simple scaling policy - add/remove N capacity units based on CloudWatch alarm - can have multiple policies (some up, some down)
  - Step scaling policy - specify one CloudWatch alarm w/ multiple steps ("add 1 when 40-70%", "add 2 when 70-90%", "add 3 over 90%")
  - Target tracking scaling - no explicit CloudWatch alarms, add or remove instances to maintain value of some metric like average CPU utilization (creates UP and DOWN alarms behind the scenes, so you can look at history of the alarm in CloudWatch)
- KEY IDEA Default cooldown - # of seconds after any scaling activity before another one can start
  - I.e. how fast to scale in or out
  - Avoids overshooting on scale in or out
  - Default of 300 sec typically good for production
  - Applies ONLY to simple scaling policy
- KEY IDEA Warmup time - how long before the metrics of a new instance will be considered for scaling policy - i.e. it takes this long for instance to start accepting traffic normally
  - Applies only to step and target tracking
  - Similar to cooldown in intent
- Can disable scale-in - never terminates instances

ASG + Load balancer integration
- Don't need a LB with an ASG, but normally do if instances are going to handle traffic
  - So instances that do standalone batch processing (timed spot, handling SQS messages, etc.) might have ASG without an ALB
- ALB has a security group (important for routing traffic to EC2)
- Create ALB -> create a Target Group (not in classic)
  - Target Group can point to instances, IP, Lambdas
  - TG specifies a health check endpoint - by default requires a 200, with default healthy/unhealthy thresholds, customizable.
- KEY IDEA By itself, a TG cannot scale! It has to be linked to an ASG.
  - TG vs ASG: https://stackoverflow.com/questions/48529074/how-is-target-groups-different-from-auto-scaling-groups-in-aws
  - Classic LB: LB -> ASG
  - ALB: LB -> TG <- ASG <--- Note that the ASG points to the TG! I.e. the ASG knows to automatically add or remove its instances from the TG!
    - But NOTE: Only adds new ones if the ASG doesn't have the AddToLoadBalancer process suspended!
  - So it's possible to create a LB with a TG not linked to an ASG; but then the TG cannot scale, it's fixed to the instances you allocate
- ELB can have access logs to S3 - disabled by default https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/access-log-collection.html
- KEY IDEA There are health checks from both ASG and LB
  - LB health checks specify whether to send traffic
    - These operate by sending traffic to the instance
  - ASG health check decides whether to terminate instances; 
    - can be one of two types: either EC2 or ELB
    - EC2 health checks: ONLY check the instance's status, healthy as long as the instance is running
    - If you want traffic-based or application checks, need to use the ELB health check
	- ELB health checks: ASKS THE ELB whether IT considers the instance healthy
	- For load-balanced instance, ELB health check is much more appropriate

ALB with HTTPS
- Want to add HTTPS to an ALB, and also redirect from HTTP to HTTPS
- Add a new listener to an existing ALB
  - First we need to create a certificate, but for that we need a domain
  - Route 53, create a domain
  - Then AWS Certificate Manager (ACM) to create a certificate
    - use server name such as app.mydomain.com
    - validates via email or DNS query
    - If DNS query, to validate the certificate, ACM checks for a specific long random CNAME under your domain
      - If using Route 53, ACM can automatically create the CNAME record
    - create DNS alias or CNAME record that points app.mydomain.com to the ALB
  - Finally, add the listener
    - HTTPS port 443 listener
	- forward to target group
	- with TLS security policy
	- point to certificate in ACM
- Also add a port 443 rule to the ALB's security group
- To do secure redirection HTTP to HTTPS 
  - edit the HTTP listener action
  - delete forward to TG
  - redirect to HTTPS with original host/path/query and 301 status
  - could do a simple URL rewrite
