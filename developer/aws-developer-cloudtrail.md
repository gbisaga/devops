### CloudTrail
- Compliance and audit for AWS accounts - history of all events/API calls within your AWS accounts
- Enabled by default (management events only)
- IMPORTANT FOR EXAM: If a resource is deleted and you want to know about it, look at CloudTrail first!

### CloudTrail vs CloudWatch vs XRay
- CloudTrail - audit API calls, useful to detect unauthorized calls or root cause of changes, delayed
- CloudWatch - all about monitoring - metrics, logs, alarms
- XRay - automated trace analysis: distributed latency, error, fault analysis
