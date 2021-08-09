Disaster Recovery
- White paper could read - summarized in lecture
- Disaster: any event w/ negative impact on business continuity or finances
- DR: preparing for and recovering from a disaster
- What kinds of DR?
  - Traditional DR on-premise - separate data centers
  - Hybrid recovery - on-premise data center with recovery in the cloud - use Route53 to direct users to one or the other
  - Cloud - region A to region B
- Two key terms - optimizing for these drives strategies
  - RPO: recovery point objective
    - Basically, how often you do backups, how far back do you go
    - This is the amount of data loss you're willing to accept - backup every hour -> you lose up to an hour of data
  - RTO: recovery time objective
    - How much time to come back - what is your downtime
  - The smaller the numbers, the more expensive

DR Strategies
- KEY IDEA Exam will give scenarios, you have to choose from these
- RPO in hours, RTO in 24 hours or less: Backup and Restore
  - High RTO, everything recreated; also high RPO because making backups takes time
  - Examples
    - On-premise: AWS storage gateway or Snowball to send to S3 - in extreme cases RPO might be a week
	- In cloud: regular snapshots into S3 of EBS, redshift, RDS, etc.
  - When you restore, recreate EC2s from AMIs, recreate RDS, etc
  - Lowest cost
- RPO in minutes, RTO in hours: Pilot Light - popular, critical systems kept running
  - Small version of the app always running in the cloud - critical core
  - Similar to Backup and Restore, but your most critical systems are already up
  - Example: Do live RDS replication to have the DB, but not running the EC2 instances
  - Lower RPO (less to backup), lower RTO (less to rebuild during recovery)
  - The difference between Pilot Light and Warm Standby can sometimes be difficult to understand.
    - Both include an environment in your DR Region with copies of your primary region assets. 
    - The distinction is that Pilot Light cannot process requests without additional action taken first, while Warm Standby can handle traffic (at reduced capacity levels) immediately
      - Pilot Light will require you to turn on servers, possibly deploy additional (non-core) infrastructure, and scale up
      - Warm Standby only requires you to scale up (everything is already deployed and running). Choose between these based on your RTO and RPO needs.
- RPO in seconds, RTO in minutes: Warm Standby - full system up and running but at minimum size
  - ASG with only one EC2 running - in recovery, scale up the ASG
  - More expensive since you have more infrastructure
- RPO near zero, RTO potentially zero: Multi-site/Hot-site - aka active/active
  - Very low RTO, but very expensive
  - Duplicate system running, Route53 sends traffic to both
  - Cloud/replica version accesses same live database as on-premise instances
  - EC2 fail over to replicated RDS slave
- All in the cloud, really same as on-premise - same options

DR tips
- Backup with EBS snapshots or backups
- HA
  - Route53 be tween regions
  - If DirectConnect and network goes down, use Site-2-Site VPN
- Replication
- Automation
  - CloudFormat/Elastic Beanstalk
  - Recover/reboot with CloudWatch if alarms fail
  - AWS Lambda for customized automation
- Testing - chaos monkeys
  - Ex. Netflix has "simian army" - randomly terminate app servers, even in production

DR checklist
- Is AMI copied across regions, with key in parameter store
- Is CloudFormation StackSet working and tested to work in another region?
- What is RPO/RTO
- Are Route53 health checks working correctly? Ties to a CloudWatch alarm?
- How can automate w/ CloudWatch events -> trigger Lambda -> perform RDS read replica promotion
- Is data backed up? appropriate for RPO/RTO? Where is it living, how synchronized and replicated?

Backups and Multi-region DR
- EFS backup options
  - AWS Backup with EFS (frequency, when, retain time)
  - EFS-to-EFS backup automation flow (now AWS Backup)
  - EFS -> S3 -> S3 Cross region replication -> EFS
- Route53 backup - no specific import/export
  - API ListResourceRecordSets for export
  - write script for imports into Route53 or another DNS provider
- Elastic Beanstalk
  - Saved configurations using eb cli
  - Can use to recreate in another region

DNS routing policies - usually part of DR strategies other than backup and restore
- Simple - direct connection to a single resource, no DR
- Failover - for active/passive failover
- Weighted - multiple resources with specified proportion; good for canary deployments
- Geolocation, latency, geoproximity - choose based on distance of client
- Multivalue - pick up to 8 healthy records at random
