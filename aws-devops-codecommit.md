There are 3 ways you can hook up automation to CodeCommit activities:
1. Notifications - SNS only - literal notification and not for taking action based on them
    Comments
    On commits
    On pull requests
    Pull request
    Source updated
    Merged
    Status changed
    Created
    Branches and tags
    Deleted
    Updated
    Created

2. Triggers - SNS or Lambda - supposed to initiate action, usually some automation
- Can also specify particular branch names (up to 10)
- Can specify "custom data" such as a chat channel name
- Events:
    push to existing branch
    create branch or tag
    delete branch or tag

3. More flexible way - see blog below - is CloudWatch events (now called EventBridge) - CW is centerpiece of all devops automations
- Specify a service name - here CodeCommit
- Then event type - commonly CodeCommit Repository State Change
- Targets - one or more - one could be CodeBuild project
- Example on "AWS DevOps Blog" - Validating pull requests with AWS CodeBuild and Lambda
    https://aws.amazon.com/blogs/devops/validating-aws-codecommit-pull-requests-with-aws-codebuild-and-aws-lambda/
- Note that each CW/EventBridge event can have multiple targets - so you can (1) add a comment to a PR and (2) start a validation build at the same time, just by creating two targets on the EventBridge event. This is one reason why these events are the most powerful and flexible way of hooking up automation.

So, we can set up notifications, triggers, and CloudWatch Rules to kick off automation

Lambda triggers - can create a trigger from the lambda side as well - e.g. to check for passwords being checked in, etc.

