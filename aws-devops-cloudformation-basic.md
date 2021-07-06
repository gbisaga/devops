Introduction - Infrastructure as Code
- Manual work hard: Reproduce in another region? Another account? Somebody deletes?
- CloudFormation is declarative way of defining AWS Infrastructure (most are supported)
- Benefits
  - No resources manually created
  - Version controlled
  - Code reviews
  - CloudFormation is free, pay for resources
  - There a stack identifier on the Infrastructure created
  - Declarative programming - order can be specified
  - Separation of concerns - separate stacks for each layer (VPC, network, application)
  - Leverage existing templates and documentation on the web
- Templates pulled from S3 - you can't edit, you re-upload template to S3
- Deleting a stack deletes every artifact it created
- Deploying templates
  - Manual through CloudFormation designer UI
  - YAML file, use CLI to deploy
- Sections
  - Resource (ONLY MANDATORY PART) - any AWS resources
  - Parameters - dynamic inputs
  - Mapping - static variable maps
  - Outputs
  - Conditional
  - Metadata
  - Helper - references and functions
- Exam is about what features to use to do X operation, and read it
- Can use 

Create a stack
- Upload template or S3
- Can provide a role or use default for login
- Rollback options
- Termination policy
- Specify notifications
- Monthly cost estimator
- Once stack running: Events list starts with CREATE_IN_PROGRESS, CREATE_COMPLETE
- Resources get tags with original stack name and unique id, plus any other tags you define (Product)

Update a stack
- Upload new one or edit in designer
- Cannot change stack name
- Preview changes; Replacement=TRUE means existing will be deleted, new one created (e.g. for an EC2 - good reason not to be a pet)

Delete a stack
- Automatically deletes everything, in correct order
- Resources can have a separate deletion policy - don't delete with the stack

Parameters
- Important for template reuse, and for parameters you can't determine ahead of time (e.g. key pair for an EC2 instance)
- Ask yourself: Is this configuration likely to change in the future? 
- Have types: `string`, `number`, `CSV`, `List<Type>`, AWS parameter
- AllowedValues or pattern, no echo (for secure parameters)
- Use !Ref (or Fn::Ref) to reference a parameter (!Ref also references name/YAML key of resources)
- Also pseudo-parameters built into CloudFormation - predefined AWS::AccountId, AWS::Region, etc.

Resources
- Key, 100s of types, AWS::ProductName::Type e.g. AWS::CodeDeploy::Application
- AWS resource type reference documentation - each key specifies if it requires Replacement, e.g. change of AZ requires Replacement
- All under Resources block, each has a (1) YAML key (the name) (2) the type (3) properties
- CANNOT have dynamic number of resources - everything declared, no code generation. CDK
- Not every resource supported, but most are

Mappings
- Fixed, hardcoding variables (dev/prod, regions vs AMIsm etc.) - very low-level
- Typical two level region + type
- Use variables when values are really user-specific
- !FindInMap [mapname, top leve key, second level key] (the brackets are literal) or Fn::FindInMap

Outputs
- Optional outputs to view
- If you EXPORT them they're available in other templates - these are a flat namespace across the entire account/region!
- KEY IDEA: lets each person develop templates according to their own expertise
- Cannot delete stack if outputs used somewhere else
- !ImportValue exportedValueName

Conditions:
- Create resources or not; example
    Conditions:
        CreateProdResources: !Equals [ !Ref EntType, prod ]
- Logical functions you can compose: AND, OR, NOT
- Apply using `Condition: CreateProdResources` in a resource definition

Intrinsic functions - KEY IDEA - MUST KNOW FOR EXAM
- `Fn::Ref` - reference a parameter or physical ID of an underlying resource (i-3423ad98f)
- `Fn::GetAtt` - get any of the attributes of a resource, not just its physical id - e.g. AvailabilityZone, PrivateIP
  - Important for example to specify the AZ of an EC2 value is the same as the AZ of its EC2 instance
- `Fn::FindInMap [ mapname, topkey, secondkey ]`
- `Fn::ImportValue exportName`
- `Fn::Join` - join with a delimiter specified \
`!Join [ ":", [ a, b, c ]]` ==> `a:b:c`
- Fn::Sub very handy - either name/value substitution or ${variableName} syntax: \
`!Sub "hello ${name}"`
- For conditions: `Fn::And`, `Fn::Equals`, `Fn::If`, `Fn::Not`, `Fn::Or`

User data 
- Need to pass entire script thru Base64
- Output user data results to local file - /var/log/cloud-init-output.log
- UserData:
    Fn::Base64 |
      line 1
      line 2
- Problem: even if user data failed, stack will say the create worked
- Better for complex scripts: cfn-init with Metadata > AWS::CloudFormation::Init
  - Have to install an agent in the userdata
  - Package install
  - Files with dynamic substition, mode
  - Random commands
  - Services to start and ensure running
  - KEY: Much more readable
  - Result goes to /var/logl/cfn-init.log 

cfn-signal and wait condition
- cfn-init still a problem because stack could complete but script failed
- Use cfn-signal script after cfn-init (in userdata)
- Define a wait condition so CloudFormation can move forward
- Specify the AWS::CloudFormation::WaitCondition that says how many signals (e.g. 2 instances), how long to wait for the signal
- Can specify bash command completion status (-e $?) or some other bash command
- The EC2 create event will happen when the instance actually created, but the cfn-signal event will still be CREATE_IN_PROGRESS
- KEY IDEA: Troubleshooting problems
  - Check AMI has CloudFormation helper scripts
  - Verify in /var/log/cloud-init.log or /var/log/cfn-init.log to debug instance launch
  - To retrieve logs from the instance you have to disable rollback on failure, otherwise the instance is deleted when the stack fails
  - Verify instance has a connection to the internet else can't talk to CloudFormation service - in VPC needs access thru a NAT device (private subnet) or Internet gateway (public)
  - OnFailure=ROLLBACK, DO_NOTHING, DELETE - in Stack Creation Options disable rollback on failure -> Get rid of by manually deleting the stack

Nested stacks
- Isolate common patterns and components, reuse in other stacks
- Examples: Load Balancer, security group re-used
- KEY IDEA: To update nested stack, always update the parent/root stack
- Invoke from parent stack with resource of type AWS::CloudFormation::Stack
- Can reference using any http reference e.g. GitHub or S3

ChangeSets
- When update a stack, want to know what will happen
- Tells you what will happen, but doesn't tell you if it will succeed
- Created - tell it to "create changeset for current template"
  - Gives you all the changes in UI (just like as part of stack execution) or JSON 
  - Useful, shows an instance will be replaced

Deletion policy
- Put on any resource or nested stack - also lets you take a snapshot
- DeletionPolicy: Retain on any instance
- DeletionPolicy: Snapshot - for EBS volume, RDS, etc. - default for AWS::RDS::DBCluster
- DeletionPolicy: Delete (default)
- Can enable delete protection on stack