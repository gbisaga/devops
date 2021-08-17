Many Source Control strategies
- Components
  - Repository - centralized place, the big container
  - Branch - collection of files with own history and lifecycle
  - Commit - single change on a given branch
  - Pull Request - merge one branch into another
- Strategies
  - Trunk-Based Development - commits directly to master, the only branch
    - Master still always releasable
    - Pro: very small changes
	- Pro: Continuous code merges
	- Pro: increate delivery
	- Con: requires lots of testing - everything needs to be releasable at any time
  - GitHub Flow
    - Master still always releasable
	- Pro: Short lived feature/bug branches - frequent
	- Con: master not always up to date
	- Con: larger changes to master
  - GitFlow
    - Pre: Master is always releasable
	- Pro: Strict control
	- Con: Master not always up to date
	- Con: Large changes to master - if something broken, have to figure out what change caused it
	- Con: Complicated merges
  - Environmental branching - one long-lived branch for each environment - master/dev, test, prod
    - Similar to GitFlow but worse - even more merging

There are 3 ways you can hook up automation to CodeCommit activities:
1. Notifications - SNS (or AWS Chatbot - e.g. slack) only 
   - literal notification and generally not for taking action based on them (old style)
     - Comments
     - On commits
     - On pull requests
     - Pull request
     - Source updated
     - Merged
     - Status changed
     - Created
     - Branches and tags
     - Deleted
     - Updated
     - Created

2. Triggers - SNS or Lambda - supposed to initiate action, usually some automation
- About code related events, to branches
- Can also specify particular branch names (up to 10)
- Can specify "custom data" such as a chat channel name
- Events:
  - push to existing branch
  - create branch or tag
  - delete branch or tag

3. More flexible way - see blog below - is CloudWatch events (now called EventBridge) - CW is centerpiece of all devops automations
- Specify a service name - here CodeCommit
- Then event type - commonly CodeCommit Repository State Change
- EventBridge also gives you source of any API call via CloudTrail
- Targets - one or more - one could be CodeBuild project
- Example on "AWS DevOps Blog" - Validating pull requests with AWS CodeBuild and Lambda
    https://aws.amazon.com/blogs/devops/validating-aws-codecommit-pull-requests-with-aws-codebuild-and-aws-lambda/
- Note that each CW/EventBridge event can have multiple targets - so you can (1) add a comment to a PR and (2) start a validation build at the same time, just by creating two targets on the EventBridge event. This is one reason why these events are the most powerful and flexible way of hooking up automation.

So, we can set up notifications, triggers, and CloudWatch Rules to kick off automation

Lambda triggers - can create a trigger from the lambda side as well - e.g. to check for passwords being checked in, etc.

Basic access
- CodeCommit is a REGIONAL service - use the region in the URL `ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/garytest1`
- Can use HTTPS or SSH or local GRC URL (git helper)

Securing repo - Restricting access
- Normally you need to give users CodeCommitPowerUser access 
- Limit access using DENY rules for particular repos or branches - have e=DENY / a=codecommit:* / r=arn:...:RepoName
- What if want to protect branch changes? Add another IAM DENY policy - nothing like S3 bucket policies
  - Add policy denying write actions like GitPush, DeleteBranch, PutFile, etc.
  - Add condition codecommit:References = refs/heads/master
- What if you want to have approvers? Add Approval Rule Template
  - Number of approvals, approval pool (user, role, ARN)
  - Specify branch filters - which are protected branches
  - Assign rule to specific repos
