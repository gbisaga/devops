# ECS
- Task is like a pod definition
- Normally define a Service linking to the Task definition
- Service type 
  - REPLICA vs DAEMON 
  - # of replicas vs one per instance. REPLICA is normal
  - DAEMON usually for per-instance monitoring task, etc.
- Minimum healthy percent - 0 for rolling
- Deployment, rolling or blue/green
- Task Placement - Strategy for assigning tasks to instances. Typically "AZ balanced spread"
- Task defs have revisions, 1-n
- ECS with load balancer 
  - use ephemeral ports, ALB with Dynamic Port Forwarding knows how to find ephemeral ports
  - To use ephemeral ports, don't specify a "host" port - just a containerPort, with hostPort=0
  - Point service to revision 2
- ECS/XRay integration - options
  - Run as one daemon task on each EC2 instance using UDP port 2000 networking
  - Run as a sidecar container - run as container in each task
  - Fargate only supports sidecar since you don't have control over instances
- set AWS_XRAY_DAEMON_ADDRESS environment variable with "xray-daemon" like "xray-daemon:2000" (and define a container link
- agent configure with with /etc/ecs/ecs.config

### EXAM QUESTION: Elastic Beanstalk 
  - in single and multi docker container mode (i.e. one or multiple containers per EC2 instance)
  - Creates ECS cluster, ECS2 instances, Load balancer (for high availability mode)
  - Task definitions and executions
  - Config file Dockerrun.aws.json at root of source code
  - Docker images prebuilt and stored e.g. in ECR
  - Or can supply Dockerfile and it will build a single image for you

### Summary + EXAM TIPS - containers, 3 flavors
- ECS Classic 
  - EC2 instances 
  - creates instances with cluster name in env and config file (ECS_CLUSTER)
  - EC2 instance special AMI with ECS agent to register instance with cluster. 
  - Can run multiple containers of same type 
    - Don't specify a host port (only container port)
	- Use ALB with dynamic port mapping
	- EC2 instance security group allow traffic from ALB on all ports
  - ECS tasks can have IAM roles - not on the instances
  - SG operates at instance level, not task level
- Fargate
  - Serverless - AWS provisions containers and assigned them ENI
  - Containers provisioned by container spec (CPU/RAM)
  - Tasks can have IAM roles to execute against AWS - uses ECS_ENABLE_TASK_IAM_ROLE config in ecs.config file
  
- ECR tightly integrated with IAM
  - always login (PUSH OR PULL) first: $(aws ecr get-login ???no-include-email ???region XXX) - generates ???docker login??? command
  - to push: docker push 1234567890.dkr.ecr.us-west-2.amazonaws.com/demo:latest/UserGuide/aws-properties-ec2-instance
  - to pull: docker pull 1234567890.dkr.ecr.us-west-2.amazonaws.com/demo:latest/UserGuide/aws-properties-ec2-instance
  - If you can???t push or pull an image, check IAM!
  
- Integrations
  - XRay - run as 2nd container within task (for fargate)
  - Ready to use AWS image
  - Set up logging at task definition level - eah container has its own log stream
- CLI
  - Create service on ECS: awc ecs create-service
  - Build an image: docker build -t demo .
