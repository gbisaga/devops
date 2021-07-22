GuardDuty
- Intelligent threat discovery to protect AWS account against mailicious usage
- ML algorithms, anomaly detection
- No software to install
- Input data
  - CloudTrail logs - API calls, unauthorized deployments
  - VPC Flow Logs - unusual internal traffic, IP address
  - DNS logs - compromised EC2 instances
- KEY IDEA Needs to create a service role for access to these three log types (CloudTrail, VPC flow, DNS)
- Continuous, Customizable
  - Integrations: 
    - Notifications and CloudWatch -> Lambda Functions
    - Sends new CWE events within 5 minutes, then update every 6 hours (configurable)
  - Can specify whitelists for IP addresses
  - Can pause and resume service
- Sample generation - RDP brute force, trojan on machine, bitcoin mining detected

Integrations
- CloudWatch events
  - Source serviceName: GuardDuty, eventType: GuardDuty Finding (no other parameterization)
  - All events are funneled thru, no filtering