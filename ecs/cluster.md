### ECS cluster setup
- Each instance runs ECS2 agent
  - Instance profile w/role
  - Contacts ECS service, CloudWatch logs, ECR service
- Tasks also have roles for S3, EC2, etc.
- Cluster has
  - Zero or more instances
  - Fargate tasks can be added at any time

### Multiple architectures
- Public subnet only
- Private/public subnet
  - NAT gateway (4c/hour)

### Task definition 
- Note that task definitions are defined OUTSIDE a cluster
- Define launch type (fargate or EC2)
  - May limit which clusters it can be run on (can't run an EC2 task definition in a fargate-only cluster)
- JSON document with
  - Image name
  - Port binding
  - Memory and CPU
  - Environment vars
  - Networking info
  - IAM role
  - Logging configuration (e.g. CloudWatch)
- Task definitions have a revision number - incremented every time
- Network mode - important
  - awsvpc - each container has a dedicated ENI (only mode available for Fargate taskdef)
    - Each has its own IP address
  - bridge - default with EC2 taskdef, containers rely on docker bridge (dynamic port mapping)
  - host - container directly uses EC2 network interface
  - none - no networking
- Task definition has one or more container definitions
  - Each container defines its image, memory limits, port mappings, many more
  - Memory limits are within the overall task definition memory - limit soft or hard (kill container if exceeds)
  - This is like a `docker run` command - creates a container from an image
  - At least one container in the task is marked Essential - must be there for task to be healthy
  - Mount extra storage
  - Can add sidecar containers - proxy, log aggregation, circuit breaking

### Roles
- Instance role - ECS service, ECR, CloudWatch

### Task
- Container port mapped to an instance port - usually don't want to map them
- ALB has dynamic port forwarding - target group knows about tasks and their ephemeral ports
- Memory and CPU limits
- Specify a task definition (not part of cluster)
- Task def launch type may limit which clusters it can be run on (can't run an EC2 task definition in a Fargate-only cluster, but can run Fargate task def in either)
- Can override lots of things
  - Environment Vars
  - All containers in task definition run on same EC2 (EC2 launch type only)

### ECS service
- How many tasks to be run
- Linked to ELB/ALB/NLb
- Can run tasks without a service, but just a one-off
