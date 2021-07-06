KEY IDEA: Need to know all the deployment modes, what they're used for - VERY IMPORTANT FOR EXAM
- Single instance - single instance, DNS name straight from instance, no ASG, etc.
- High availability - ASG, multiple AZs, ELB DNS name
- Great summary: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

Updates
- All at once - fastest, instances not available for a bit (downtime)
- Rolling - update a few instances at a time (bucket)
- Rolling with additional batches - spin up new instances to move the batch
- Immutable - new instances in a new ASG, swap out whole ASG when all is healthy

All at once
- Stop all the instances, deploy new versions to all the instances
- Fastest to deploy
- No additional costs

Rolling
- Runs below capacity for awhile
- Set the bucket size - how many instances will be updated at a time
- So e.g. if 4 instances, with bucket size = 2 we have 1/2 capacity during the update
- After first bucket, move on to the next bucket
- Can take a long time to upgrade if bucket size small vs total # instances
- Note: tradeoff of deploy time vs. reduction in capacity

Rolling with additional batches
- Zero downtime
- Never reduces capacity
- KEY IDEA: Some additional cost because you temporarily leave the old ones in place
- Good option for zero downtime production
- Note that the "additional batch" instances don't stay around - terminated at the end

Immutable
- Zero downtime
- New instances on a temporary ASG that are moved to the original one when done
- High cost, double capacity
- Positive: very fast rollback - just deletes the new instances
- (1) create in temporary ASG (2) move to main ASG (3) terminate original instances

Blue/Green - is there blue/green? not a direct feature of EB, but you can make it happen
- https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.CNAMESwap.html
- With all the above options, no chance to do manual verification of environment (although can have automated) or manual deployment approval
- Create a separate "stage" environment
- Validate the stage environment
- Route 53 set up to migrate canary traffic
- When done, use EB to swap URLs 
  - EB "swap URLs" command literally and immediately swaps the URLs (CNAME) of two environments
  - Maybe better would be to use Route53 directly, and do % traffic in old vs new

Two kinds of rolling update configs:
- Application deployments and configuration updates
