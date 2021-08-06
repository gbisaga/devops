To read up on:
- CloudWatch events, metrics, alarms
- Cognito identity pools, sync
- CodeDeploy
- SAM
- API Gateway
  - https://www.alexdebrie.com/posts/api-gateway-elements/
	
Downloads: https://courses.datacumulus.com/certified-developer-k92

 
Overall EXAM tips - https://aws.amazon.com/certification/certified-developer-associate/
- AWS architecture best practices - ALBs, ASGs
- Shared responsibility model - AWS take care of security of infrastructure, but creating SGs properly, enabling encryption, etc. is your responsibility - AWS not guarantee application will be secure. Just that AWS services won't be hacked.
- Cloud-native = Lambda or ECS/Docker
- Containers in development process - ECS=Docker=Scalable model
- Tips
  - Practice makes perfect - do some AWS practice - use roles, deploy applications, etc.
  - Go thru course content one more time
  - Practice ideas 
    - take one of your existing apps, deploy on EC2 manually
	- then use EB and have it scale
	- try CICD pipeline for it
	- try decoupling components and using SQS/SNS
	- try running it on Lambda and other serverless friends
	- write automation scripts to e.g. (1) shut down EC2 instances at night; (2) automate snapshots of EBS volumes (3) list under-utilized EC2 instances (CPU<10%)
  - Questions
    - For all questions, first rule out answers you know are wrong
    - For remaining answers, figure out which makes the most sense. 
	- Very few trick questions. If it seems possible but very complicated, it's probably wrong. Might be two possible answers, one hard and one easy.
	- Questions usually have key words (e.g serverless = API Gateway + Lambda + DynamoDB)
- Skim through whitepapers - they're long. Everything is covered.
  - Security best practices
  - Well-architected framework and Architecting for the Cloud
  - CI/CD
  - Microservices on AWS - Lambda and ECS and decoupling them
  - Serverless architectures with Lambda
  - Optimizing Enterprise Economics with Serverless = key points (1) don't maintain (2) only pay if you use them
  - Containerized microservices - use Docker and ECS, only very high level
  - Blue/Green deployments - EB, Lambda, API Gateway - testing new (green) version while current (blue) is still running.
    --> Probably most important
  - Read the FAQs - often on the exam
- Get into the AWS community - talk to other Amazon engineers
  - Q&A - help others, review other people's questions
  - Forums, blogs, meetups
  - Re:invent videos on YouTube
- 2 minutes / question (65 questions in 130 minutes)
