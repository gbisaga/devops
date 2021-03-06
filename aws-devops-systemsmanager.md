# AWS Systems Manager
- Manage EC2 and on-prem at scale
- Operational insights on state of infrastructure
- Detect problems
- KEY IDEA Patching automation for enhanced compliance
- Windows and Linux
- Tightly integration with CloudWatch and Config
- Features
  - Resource groups
  - Insights
  - Compliance
  - Parameter store
  - Automation (start/stop EC2s)
    - Run command
  - Session manager
  - Patch/maintenance manager
  - State manager

### Pieces
- SSM - AWS
- SSM Agent - part of Amazon Linux 2 and some Ubuntu
  - EC2 instances - easy, just needs agent and IAM role, automatically registers
  - KEY IDEA - If not controllable with SSM, probably no agent, problem with agent, or instance doesn't have IAM role for SSM
  - Also "hybrid activation" - lets you have on-prem or other servers
  - EC2 instance id always starts with "i-XXX" - hybrid start with "mi-XXX" (can pretend by not giving it the IAM role)
  - Can even tag on-prem instances in SSM
  - Download and install the SSM agent; register agent with activationCode, activationId, and region
    - A given activationCode/activationId has registration limit - max # of on-prem instances that can use it
- SSM lists managed instances - if has running agent with role, they show up automatically
- Home regions - collects from others

### Resource Groups
- Group resources either by tag or CloudFormation stack based
- Not automatic like Azure - but is dynamic
- What can you do with resource groups? Run commands!

### Automation - Run command
- Large number of standard "documents" - either supplied by Amazon or created by me/shared with me
- Groups Command, Automation, Policy, Session
- Examples
  - AWS Update SSM Agent - update agent to latest or specified version
  - AWS-RunShellScript (or remote, salt, ansible, etc.)
- Each has parameters, max execution time
- Run by: instance tags, manual instance selection, use resource group
- Rate control - number of targets at time, or percent of instances
- Error threshold - how many errors (or percent) before you fail it
- Note: only runs on managed instances - so you can't find out instances that aren't

### SSM Parameter store
- Enter a free custom name 
  - Typical /my-app/dev/db-url - name of app + environment + parameter name
  - Important format for get parameters by path, recursive
- Type String, StringList, SecureString (use a KMS key)
- Each parameter is versioned - audit trail with version number, date, and who
- Can protect params with IAM individually
- API calls to retrieve multiple parameters (by name or path), ask for decryption --with-decryption
- Also public parameters
  - Region specific AWS parameters, grouped by service
  - E.g. latest Amazon Linux 2 AMI
  - Better than baking into CloudFormation mapping section
- CloudFormation integration
  - CloudFormation automatically recognizes when you use a "AWS::SSM::Parameter::Value<XXX>" parameter
  - If UsePreviousValue/Use Existing Value option used, this refers to the name, not the value
  - I.e. stack updates will always consider the value of the SSM parameter to decide if changed
- KEY IDEA - simplify architecture, centralize configuration storage
- Q: How would

### Patch Manager
- Patch baselines - set of default baselines created, one per operating system
- KEY IDEA Create own baselines - which patches will be applied, where to get them from if not standard location
- Custom baselines can be set as default for this OS
- Can be autoapproval after release of a patch
- Patch exceptions 
  - set a list of approved and rejected patches
  - package name formats with CVE ID, full package name with dates and versions, etc
  - So: you can approve/reject certain specific patches, certain categories, certain bugs fixed, etc
- KEY IDEA Patch sources - important bit about custom baselines, lets you specify your own patch source
- Use maintenance window for when to apply patches
- Tracks overall compliance of instances - where they are against baselines
- Can do actual install, or scan for compliance
- Tracks history of patches

### Maintenance windows
- Window says how often, what time of day (schedule cron builder, rate, cron)
- Specify "target group" - instances to be patched - just like run command, by tags, manual instances, or resource group
- Specify "tasks"
  - Typically a run-command - same as documents used for "run command" - AWS-RunPatchBaseline is common
  - Can also use Automation documents or custom tasks (Lambda or Step function)
- Also have rate control and error threshold from run-command
- Can specify multiple tasks for a window, not just a run command - also automation tasks, Lambda, or Step Function tasks
- Example: Generate an AMI by choosing the Automation document `AWS-CreateImage`

### Inventory 
- Collect list of configurations from all instances tracked by SSM
- Run periodically pulling info from instances (specified by tag, individually, resource group
- Many possible - OS versions, top applications running, AWS components (agents), network config, windows update, instance detailed info
- Also pulls specific files or windows registry values

### SSM Automation
- Simplifies common maintenance and deployment atsks of instances and other resources
- Kind of like CloudFormation with JSON 
- Build workflows to configure and manage
- Predefined AWS workflows or custom
- Notifications or CloudWatch events - automate based on automation results
- KEY IDEA Use cases - make automation tasks much simpler
  - Use stop instance with approval to require approval
  - Approval on restart
  - Create golden AMIs from source AMIs
  - Recover impaired instances, unreachable due to network, RDP, firewalls without manual steps
- Example: launch instance, update SW, stop instance, create image, terminate instance -> output fully patched AMI
- Has an automation document - big JSON with parameters, each step to perform
  - Two kinds of documents - automation and (run) command/session
  - Shows all the steps into the UI 
- KEY IDEA Can use CloudWatch events - source = SSM > Automation > Status change -> targets = run Lambda, run command, Inspector assessment template, etc.
- KEY IDEA White paper "building AMI factory process" - 4 phases
  - now recommends AWS Image Builder
  - ???obsolete??? https://d1.awsstatic.com/whitepapers/aws-building-ami-factory-process-using-ec2-ssm-marketplace-and-service-catalog.pdf
  - Build phase - above automation - base AMI -> EC2 -> Update/patch -> Golden AMI
  - Validation phase - build EC2 from it, verify with scripts, services, or Amazon Inspector
  - Approval phase - approve, put golden AMI id into SSM parameter store
  - Notifications - SNS, CloudWatch, email notification

### SSM Session manager
- Similar to "instance connect" under EC2 - terminal window
- Go to EC2 or on-prem managed instance (EC2 instance connect doesn't have)
- In addition to instance connect, keeps a session history - send session history and results into S3 or CloudWatch logs
- can tunnel ports (no logging)

### SSM Distributor
- Install packages on all managed instances (rpm, yum, windows, etc)
