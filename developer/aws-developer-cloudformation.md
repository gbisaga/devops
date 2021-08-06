CloudFormation - FAQ
- Can't create dynamic amount of resources. Everything has to be declared in a template. No code generation.
- Almost every service supported, a few niche that are not yet. Work around using Lambda Custom Resources.

Parts of template:
- CloudFormation Parameters - important for reuse or inputs you can't determine ahead of time.
  Ask yourself if it's likely to change in the future. Won't have to re-upload a template to change its setting value.
- Type: string, nuber, comma delimited list, List<Type>, AWS Parameter, description, constraints like values/length/allowed values/regexp
- Fn::Ref or !Ref - reference a parameter or a resource with the name (i.e. a key under Parameters or Resources section)
  Also can ref pseudo parameters - AWS::AccountId, AWS::NoValue, AWS::Region, etc.
- Mappings - fixed variables in template. Good for different environments, regions, AMI types, etc. Simple two level map. Useful when you know in advance all the values that can be used. Allow safer control over the template - not hardcoding in the body.
  Use Fn::FindInMap [ MapName, TopLevelKey, SecondLevelKey ] = e.g.
  !FindInMap [ RegionMap, !Ref "AWS::Region", 32 ]
- Outputs (IMPORTANT) - optional, to import into other stacks. View outputs in console or CLI - easily retrieve values.
  Cross stack collaboration, let the expert handle their own part of the stack. 
  NOTE: Can't delete a stack with outputs referenced somewhere else. If don't use Export+Name (optional) can't access in another stack.
  Reference: !ImportValue exportedName
- Conditions block - control creation of resources or outputs based on conditions. Such as in dev/test/prod, based on region, param value; can compose.
  Conditions:
    CreateProdResources: !Equals [ !Ref EnvType, prod ] 
  Logical id (CreateProdResources) is up to you. Apply this in resources using the "Condition" clause, same level as type, etc:

With or without replacement/interruption - shows in ChangeSet

  Resources:
    MountPoint:
	  Type: "AWS::EC2::VolumeAttachment"
	  Condition: CreateProdResources

Intrinsic functions - many but a few must-know ones
- Fn::Ref - most important, reference parameter (value) or another resource (returns physical id, e.g. for EC2 i-234234d234)
- Fn:GetAtt - attributes attached to any resource you create - see docs e.g. https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html#aws-properties-ec2-instance-return-values
  E.g get AZ of EC2: !GetAtt ec2InstanceName.AvailabilityZone
- Fn::FindInMap [ MapName, TopLevelKey, SecondLevelKey ]
- Fn::ImportValue exportedName
- Fn::Join - !Join [ ":", [ a, b, c ] ]
- Fn::Sub - substitute values, use ${...}
- Condition functions - And, Equals, If, Not, Or, etc.

CloudFormation Rollbacks
- If it fails, everything will be rolled back. On creation, can disable rollback to troubleshoot what happened.
- On Update, automatically rolls back to last known working state. See in log what happened, error messages.
