Cost allocation tags
- Many kinds of taags
- To show up in detailed costing reports -> mark tags as Cost Allocation Tags
- Two types of cost allocation tags
  - AWS generated
    - Automatically created by AWS e.g. `aws:createdBy`
    - Won't be applied to resources before activation
  - User tags
    - Defined by user
	- Mark them for use with cost allocation reports
	- E.g. `Environment` so you can track and filter reports by your environment level
- Takes up to 24 hours to show up in the reports

Billing and Cost Management console
- Set up report with Cost Allocation Tags item
- AWS tags don't show up on resources themselves, they only show up in reports
- KEY IDEA If you want to figure out your costs or control your budget by environment, use Cost Allocation Tags
