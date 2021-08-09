Thoughts for during the exam
- Read the question carefully for keywords like "logs", "real-time", "cost effective", "cross-account", "steps", "visualize", "correlate" (dashboard)
- To ensure we get to all the easy answers, 
- Write down all numbers, underlining ones not 100% sure of (double underlining for a guess), and review those first. Check off as they're reviewed and fixed.
- On review, if it's not clear which answer is right, don't guess except as a last resort. Analyze how answers are same or different. If two answers refer to one service and three to another, does one match the number of answers you have to choose?
- If answers are long, find common elements between them (often they will be the head of the answers). This lets you find the differentiators between right and wrong.
- Look for "only" in answers. Is it ONLY true there, or are there other options?
- It may be more than one answer that will meet the objectives. If an answer is convoluted, it's *probably* not right, but it might be. Usually common situations have straightforward solutions.
- If an answer has something you've never heard of in studying, it's probably wrong.
- If the question is complex, the common parts and differentiators of the answers can help weed out what parts of the question to ignore and what to pay attention to.
- In a lift-and-shift, look for older technologies and AWS services that would replace them. E.g. generating a tape shipped offsite -> S3 with a lifecycle policy that goes to glacier
- Compare time-based elements in questions and answers. So if the question says "hot standby", "real-time", etc., it would not include scheduled actions.

Before the exam
- Review all the "KEY IDEA", "NOTE", and "??" items
- Read blue/green https://d0.awsstatic.com/whitepapers/AWS_Blue_Green_Deployments.pdf (this should help the deployment modes table)
- Find all the services with notifications and list together in a document
- Find all the deployment modes (blue/green, all at once, rolling, etc) and list in a table against their services
- Deployment services (OpsWorks vs CloudFormation etc.) - https://cloud.in28minutes.com/aws-certification-elastic-beanstalk-cloud-formation-opswork-codedeploy-differences
- Read up on AWS Config - what are standard rules and remediations
- How can I understand everything cross-account? E.g. cross-account log destinations https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CreateDestination.html
