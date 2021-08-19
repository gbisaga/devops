# Self-healing
- OpsWorks has self-healing in a layer
  - Checks instances
  - If not healthy for ~5 minutes, it recreates them
- ASG
  - Scale instances based on CloudWatch metrics
  - Also can simply be used to replace existing ones when unhealthy
- RDS - Multi-AZ
