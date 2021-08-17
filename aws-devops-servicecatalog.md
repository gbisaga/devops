# AWS Service Catalog
- KEY IDEA: "self-service portal" that allows users who aren't AWS experts to launch predefined products (CloudFormation templates) created by adminitrator, e.g. specific EC2 configs, DB config, etc
- Restricts options for users
- Products and Portfolios
  - Admin creates products (stacks, DB, EC2, etc.)
  - Each "product" is a CloudFormation template
  - Then assign products to portfolios or catalogs
    - Catalogs are only current products, portfolios include a catalog plus past and future
  - A "portfolio" is generally like a team - UI designers, compliance managers, etc
  - Need to assign users, groups, roles to use the portfolio
  - Products have a bunch of input parameters - instance name, DB URL/password, SSH location, etc.
- Users bring up product list, restricted by IAM - access to launching products without deep AWS knowledge
- Helps with governance, compliance, consistency
- KEY IDEA Integration with other self-service portals like ServiceNow
- Brand it as your portal - title/logo, colors, etc 
- Launch product: enter parameters, get stack outputs shown in Service Catalog

### Constraints
- Various Config types for a product in a portfolio
  - Launch - role assumed by service instead of user’s
  - Notification - SNS topic
  - Tag update - whether end users can apply custom tags
  - Stack set - choose accounts/regions/role to create stack set

  
