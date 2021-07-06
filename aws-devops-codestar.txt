Quickly develop, build and deploy applications in AWS - integrates all the pieces
- KEY IDEA: customize included template.yml file in repo to customize CodeStar behavior
- Choose a template - static, node.js application
- Creates a repo (contains appropriate sample code), CodeBuild with a buildspec.yml, CodePipeline to build and deploy
- Creates and executes CloudFormation to provision the resources
- Integrates with jira to track issues
- CodeStar builds a repo with the usual files (e.g. buildspec.yml) appropriate to the CodeStar template you use
  - But adds template.yml - a CloudFormation template that specifies CodeStar parameters
  - You can customize template.yml
- Manages Cloud9 web IDE
