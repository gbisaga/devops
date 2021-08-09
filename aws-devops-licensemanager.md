LicenseManager
- Manage licenses (Microsoft, Oracle, etc.)
- Define rules for licenses, attach to resources - you specify how many licenses you have, it tracks how many you're using
- Need to grant permissions to LicenseManager
- Counting model - by instance, vCPU, core, AMI
  - Note that this is best way especially for per-vCPU licenses. You can count EC2 instances, but it's not easy to count vCPUs.
- AMI - every time an instance is created with this AMI, it uses up a license
