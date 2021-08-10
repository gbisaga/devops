### Software delivery technology
- Apps packaged in container run on any OS, any language, any technology
- Predictable behavior, no surprises
- Containers don't impact each other, but can be linked, networked together
- Repos - public (docker hub) or private (ECR)
- Vs a Virtual Machine - sort of virtualization technology
  - Resources shared with the host
  - With VMs, there's a hypervisor plus Guest OS's
  - Just a docker daemon and containers - much lighter weight
  - Result: Can have lots more containers running than VMs

### Also need container management platform - AWS has three
- EKS - not in exam
- ECS
- Fargate

### ECS details
- Cluster - a logical grouping of EC2 instances
  - run a special Amazon AMI
  - ECS agents register to the ECS cluster - agent needs to be running (diagnostic)
  - provisioning model with on-demand or spot instances
  - an ECS instance can hold multiple task instances that share a certain amount of CPU and memory
  - a cluster has an ASG (multi-AZ) and one or more instances
  - you can scale a cluster by the ASG normally
  - ECS Instance role is assigned to the EC2 - among other things to pull images from ECR, register with ECS, logging; user data sets up to register with cluster; there's an always-running container that actually registers with ECS
- ECS task definitions
  - metadata in JSON form
  - one or more containers
    - each has info on image name, port bindings and mappings (host port to container port), memory/CPU, environment vars, etc.
	- main and sidecar containers
  - KEY IDEA: tasks can be assigned a role, very important for tasks to do anything
  
### ECS service
- manages running tasks, built from task definitions
- define how many copies of task definition should run and how - ensure number of tasks desired is running across ECS instances
- can be linked to ALB/ELB/NLB if needed - not necessary but obviously want to if you have a webapp
- two kinds of service: REPLICA (running n copies of tasks somewhere) or DAEMON (one copy on each instance)
- 2 deployment types - rolling or blue/green (CodeDeploy)
- KEY IDEA: note that if you use a port mapping, only one task can run on each EC2, since it binds a host port; but if using a load balancer can do dynamic port forwarding (ALB only) to route traffic to ephemeral/random ports
- load balancer - can't add to an existing service, only during create. Create the ALB first then assign to service.
- KEY IDEA: EC2's SG needs to allow the ALB to talk to any EC2 port

### ECR - private docker image repository
- KEY IDEA: Access controlled through IAM - so if instance can't pull images from ECR, probably instance role doesn't include ECR permissions (the auto-generated role will but you could manually change this)
- docker push and pull 
  - KEY IDEA: first make sure to do $(aws ecr get-login ...) - need right IAM permissions
  - build image
  - tag image to include the repo name
  - docker push full-image-name
- docker pull
  - login
  - docker pull full-image-name
	- same image name, in form repo/imageName:version (e.g. latest)
  - to pull from ECR, update (1) task definition to use full repo image name (2) service definition to refer to new task
- for a rolling update, will be a mix for a period of time
- will take a while because it allows a period of time (default 5 minutes) for draining time 
- can manually kill the tasks

### Fargate
- Without it you have to create EC2 instances, scale them via ASG
- With Fargate, no EC2's created: just create task definitions, and ECS run containers
- To scale, just increase the number of tasks
- No host mapping, will do dynamic routing on its own

### Elastic Beanstalk and ECS
- KEY IDEA: EB runs docker in single container or multi-container docker mode
- Multi-container mode creates an ECS cluster
- KEY IDEA: Config file Dockerrun.aws.json at the root of EB source code
- Docker images pre-built and stored in ECR (or other repo)
- EB doesn't define ECS services - it defines task definitions and tasks directly

### IAM roles - KEY IDEA: 2 kinds as regards ECS, instance and task
- 1) IAM role attached to the EC2 instances as above - ECS, ECR, logging - need this just to run tasks in ECS
- 2) IAM role assigned to individual tasks - required if tasks are going to do things in AWS (e.g. read/write S3)
- With Fargate, only the task role (if at all)

### ECS task autoscaling
- On ECS service definition, specify an autoscaling policy
- Autoscale number of tasks based on a policy - just like an ASG - 2 kinds
  - Target - scale up or down based on some metric, e.g. ECSServiceAverageCPUUtilization
  - Step - scale up or down based on an alarm - more complicated, alarm on metric X, for more than one minute add 2 tasks, etc.
  - Target is much simpler
- NOTE: Service autoscaling autoscales tasks but NOT EC2 instances! So it's possible to autoscale with classic ECS but you have to autoscale both tasks and instances separately. So:
  - Fargate works much better, just autoscale the service
  - KEY IDEA: If you want to autoscale a classic ECS cluster, use EB that will scale both together
