### template parameters
- string, number, comma delimited list
- many special AWS types such as AWS::EC2::Image::Id, AWS::EC2::Instance::Id, AWS::EC2::VPC::Id, List<AWS::EC2::VPC::Id>
  - https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html#aws-specific-parameter-types
- Can pull parameters from SSM Parameter Store
  - Good for a default central company-wide AMI or other similar policy value, neither hard-coded nor user-entered
  - Need to use special Type under AWS::SSM::Parameter
  - Can't use SecureString parameters
    Parameters:
        InstanceType:
            Type: 'AWS::SSM::Parameter::Value<String>'
            Default: /EC2/InstanceType <-- Says the default parameter values from there, or the default keys in the store?
- There are also "public parameters" from AWS - e.g latest AMI for certain combinations, e.g. /aws/service/ami-amazon-linux-latest/XXXX

### Other special template properties
- `Code`: CodeCommit initial contents - ZIP file in S3 (e.g. README.md file)
- DependsOn: otherResource
  - Tells one resource not created until another resource created - normally they are created in parallel where possible
  - Good e.g. for an application starting in the app that wants to contact a DB created in another resource
  - Also specifies the deletion order

### Lambda functions
- Two ways of defining
    - Can define inline - no dependencies and limited to 4K characters, good for small functions
    Code:
        ZipFile: |
        # Code goes here
    - Can zip the function and its dependencies in a zip file and upload to S3
      - How do you redeploy? Template doesn't change, S3 bucket and key name don't change 
      - enable S3 versioning and add new S3ObjectVersion in the Code: block
      - This isn't automatic though, you have to have an explicit parameter that you update it
- Creates IAM role - CloudFormation requires you to say OK (capability)
- Lambda function is linked to CloudFormation, shows on Lambda console

### Custom resources
- KEY IDEA: A few key use cases
  - New AWS resource not yet handled
  - Managing on-premise resources
  - Emptying S3 bucket - because bucket created in stack can't be deleted with objects in it
  - Fetch an AMI id
- Events are passed, only called if there is Create/Update/Delete event, NOT every time you run the template <- When are these?
  - RequestType, ResponseURL

### Drift
- Manual intervention
- Need to manually check for Drift
- Not all resources support drift checking
- Results: ADD (e.g. SG rule), CHANGED values (e.g. IP address)
- Cannot revert back, just for detecting changes

### Stack status codes
- KEY IDEA FOR EXAM: UPDATE_ROLLBACK_FAILED - unsuccessful return of any resources 
  - Some common cases: failed to receive number of signals, changes outside CloudFormation, insufficient permissions, invalid security token
  - can only delete the stack or continue the rollback
  - before had to delete or call AWS support
  - now, some cases can be continued after fixing permission or limit
  - fix then “continue rolling back” command
  - AWS DevOps blog "continue rolling back an update for UPDATE_ROLLBACK_FAILED" https://aws.amazon.com/it/blogs/devops/continue-rolling-back-an-update-for-aws-cloudformation-stacks-in-the-update_rollback_failed-state/
  - Online docs helping to troubleshoot

- CREATE_COMPLETE (green)
- CREATE_IN_PROGRESS (blue)
- CREATE_FAILED - understand what went wrong
- DELETE_COMPLETE - everything OK
- DELETE_FAILED - maybe something still running, S3 bucket with contents - try delete again
- DELETE_IN_PROGRESS
- ROLLBACK_COMPLETE - 
- UPDATE_COMPLETE, UPDATE_COMPLETE_CLEANUP_IN_PROGRESS
- UPDATE_IN_PROGRESS, UPDATE_ROLLBACK_COMPLETE, UPDATE_ROLLBACK_COMPLETE_CLEANUP_IN_PROGRESS

### InsufficientCapabilitiesException
- KEY IDEA: These aren't IAM permissions or a problem with your template, e.g. CAPABILITY_IAM
- You get these when you don't check the checkboxes "agree to create IAM roles", etc.
- InsufficientCapabilitiesException is the error message you get back from AWS in the API

### cfn-hup
- Checks for changes to the metadata periodically
- KEY: Default interval=15 minutes, but can change in template
- Can be used to handle updates to metadata - such as changes a value in the index.html (Goodbye -> Goodbye again)

### Stack policies
- Looks like an IAM policy, lets you deny updates on certain resources e.g. CriticalSecurityGroup or any ResourceType=AWS::RDS::DBInstance
- You can update the stack's policy to allow it, making the conscious decision; but KEY IDEA that it's a temporary change
