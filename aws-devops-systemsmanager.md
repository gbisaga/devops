AWS Systems Manager
- Manage EC2 and on-prem at scale
- Operational insights on state of infrastructure
- Detect problems
- KEY IDEA Patching automation for enhanced compliance
- Windows and Linux
- Tightly integration with CloudWatch and Config
- Features
  - Resource groups
  - Insigheds
  - Compliance
  - Pamarmeter store
  - Automation (start/stop EC2s)
  - Run command
  - Session manager
  - Patch/maintenance manager
  - State manager

Pieces
- SSM - AWS
- SSM Agent - part of Amazon Linux 2 and some Ubuntu
  - EC2 instances - easy, just needs agent and IAM role, automatically registers
  - KEY IDEA - If not controllable with SSM, probably no agent, problem with agent, or instance doesn't have IAM role for SSM
  - Also "hybrid activation" - lets you have on-prem or other servers
  - EC3 instance id always starts with "i-XXX" - hybrid start with "mi-XXX" (can pretend by not giving it the IAM role)
  - Can even tag on-prem instances in SSM
  - Download and install the SSM agent; register agent with activationCode, activationId, and region
    - A given activationCode/activationId has registration limit - max # of on-prem instances that can use it
- SSM lists managed instances - if has running agent with role, they show up automatically
- Home regions - collects from others

Resource Groups
- Group resources either by tag or CloudFormation stack based
- Not automatic like Azure - but is dynamic
- What can you do with resource groups? Run

Run command
- Large number of standard "documents" - either supplied by Amazon or created by me/shared with me
- Groups Command, Automation, Policy, Session
- Examples
  - AWS Update SSM Agent - update agent to latest or specified version
  - AWS-RunShellScript (or remote, salt, ansible, etc.)
- Each has parameters, max execution time
- Run by: instance tags, manual instanace selection, use resource group
- Rate control - number of targets at time, or percent of instances
- Error threshold - how many errors (or percent) before you fail it
