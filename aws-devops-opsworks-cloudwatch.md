### Integrated with CloudWatch events
- Option Auto-healing enabled
- Instances have an agent to communicate with service to restart - this happens automatically
- Q: How do you get notified?
  - Create CloudWatch event, source=OpsWorks > Instance state change -> target=SNS topic, lambda, etc.
  - If you want to know specifically when change due to auto-healing, filter on { detail: { initiatedBy: [ "auto-healing" ]}
  - also “auto-scaling”