WAF - Web Application Firewall
- Implements checks like OWASP Top 10 (SQL Injection, XSS, etc.)
- Create Web ACLs - for each
  - Specify a CloudWatch metric (why??)
  - Specify zero or more resources
    - API Gateway
    - ALB
    - AppSync
- Add rule sets
  - Custom rules using pattern matching (query string looks like/not like this)
  - AWS Marketplace has rule sets like OWASP

Shield - Managed DDoS protection
- 