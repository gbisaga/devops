# Multi-AZ
- Sometimes must be enabled manually
  - EFS, ALB/NLB (attached to a VPC, can specify subset of AZs), ASG, beanstalk: assign the AZ's (multiple AZs)
  - RDS, ElastiCache: multi-AZ (synchronous standby DB for failover)
  - Aurora - data stored automatically across multi-AZ, but can have multi-AZ for DB (like RDS)
  - ElasticSearch (managed): multi-master
  - Jenkins (self deployed): multi-master in an ASG
- Implicitly
  - S3 (except OneZone-IA)
  - DynamoDB - always replicated across AZs
  - All of AWSs proprietary services - automatically
- EBS - special case
  - Tied to a single AZ (obviously, it's just a disk)
  - How to make multi-AZ? Use automation!
    - Create ASG with 1 min/max/desired - ASG is so a new EC2 will be automatically created
    - Lifecycle hook for terminate: make snapshot in S3
    - Lifecycle hook for start: copy snapshot, create EBS, create EC2 instance attached to it
    - NOTE: for copy, pre-warm PIOPS (provisioned IOPS, "io 1") volumes, need to read entire volume once - pre-warm the blocks

### Multi-region services
- New, sometimes need to manually implement - be creative with automation
- DynamoDB global tables (multi-way replication, enabled by Streams)
- AWS Config Aggregators (multi-region, multi-account) - I did this!
- RDS cross region read replicas (used for read and DR)
- Aurora "global" database
  - Up to five secondary regions
  - One region is master, secondary regions for read and DR
  - Easy to "promote" replica to master (RPO 1 second, RTO 1 minute)
- EBS volume snapshots, AMI, RDS snapshots can be copied to other regions
  - Typical to put name in parameter store so easily reference correct one for region
- VPC peering - Private traffic between regions
- Route53 - always multi-region, global network of DNS servers
- S3 cross-region replication - good for DR, low-latency in another region, aggregate for analytics
- CloudFront for global CDN at all edge locations (many, more than # regions)
- Lambda@Edge for global Lambda function at all Edge locations

### Multi-region with Route53
- KEY IDEA Health check is center of it -> automated DNS failovers
  - Two ALB+ASG in different regions
  - Route53 record directs users with traffic criteria (latency, geoproximity, etc.)
  - BUT we need to make sure the region is healthy -> so add Health Checks
  - Three ways to set up health check in Route53
    - https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-determining-health-of-endpoints.html
    - Monitor an endpoint - can be good, but it's very simple (i.e. app is reachable)
      - multiple checks must be satisfied including status, SearchString, response time (4 sec), 18% globally
	- Health check that monitors other health checks ("calculated" health check)
	  - up to 256 endpoint and alarm checks
	  - AND, OR, X OF Y, NOT conditions
	- KEY IDEA Monitor any CloudWatch alarm (backed by any CloudWatch metric) 
	  - full control, any condition: DynamoDB throttling, CPU, custom metrics, etc.
	  - evaluates data stream, not alarm itself
- Health checks generate CloudWatch metrics - so you can notify based on the Health check results
  - e.g. SNS notification

### Multi-region CloudFormation StackSets: Deploying an application across multiple regions and accounts
- Create/update/delete stacks across multi-account, region, organizational units
- KEY IDEA When you see "global deployment" think StackSets and CodePipeline (see below)
- Two user/roles involved: Administrator to create StackSet, trusted account to modify and delete (I don't understand this???)
- When you update a StackSet, all associated stacks are updated
- Can set maximum concurrent actions (how many account/regions at a time, # or %), failure tolerance before rollback (# or %)
- Can add new stack instances or delete instances to existing StackSets
- Any CloudFormation template will work, but oriented around enabling and configuring things
  - Enable CloudTrail, AWS Config, GuardDuty
  - Add common AWS Config rules
- Delete StackSet - have to delete all the stack instances first

### Multi-region CodePipeline
- KEY IDEA Do multi-region deployment built into CodePipeline - https://aws.amazon.com/blogs/devops/using-aws-codepipeline-to-perform-multi-region-deployments/
- CodePipeline in one region invokes CodeDeploy locally but also in other regions
- Artifact store (versioned S3 bucket) needs to be created in the other regions so CodeDeploy can access them
- CodePipeline has specific Action steps for each region - specify the region in each action
