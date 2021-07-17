CloudWatch alarms
- Create alarms based on metrics
- Each alarm has a state: OK, Insufficient data, In Alarm
- Alarm based on any metric, including custom
- CANNOT have multiple metrics in one alarm
- Choose a statistic and a period
- Conditions
  - Static - greater/less that a certain value
  - Anomaly detection - inside or outside a band
  - Can say X of Y must breach
  - How to treat missing data (breaching threshold, not breaching, etc.)
- Actions - what happens if alarm state
  - Only target = Create one or more SNS notifications
  - Can also create autoscaling action, for an ASG or ECS cluster
  - For EC2 metrics only: also special category of EC2 actions - start, terminate, reboot, etc.

View alarm
- Status - starts out in Insufficient Data
- History - status transitions
- KEY IDEA Note: CloudWatch alarms is NOT an event source in CloudWatch Events!
  - Only notification you can do is send to SNS topic
  - Plus autoscaling and EC2 actions

Billing alarms
- Only in us-east-1
- Set threshold normally, send notification when you spend more than (e.g. $10 in a 5 hour period)
- Or cumulative
