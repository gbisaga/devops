Really, "everything is automation"
"The “as code” buzzwords started with “infrastructure as code” and the DevOps movement, when IT operations/sysadmins and developers started working together to automate IT environment modifications with reusable code and then version control that code much like developers had been handling their application code changes for many years before."

From the SRE book https://sre.google/sre-book/introduction/

# IaC - Terraforming the datacenter

## 1. Goals
- Unify view of resources
- Expose way for individuals and teams to safely and predictably change infrastructure
- Workflow that is technology agnostic - less about tooling, more about a workflow
- Manage anything with an API

Ops: "I don't know how to code. But I do want to automate how I configure my infrastructure."
Devs: "We don't want to mess with infrastructure. It's not testable, not S/W development practices."

HCL - Intent-oriented language - not learning to code. Specification aligns with data model.
- Not common across providers, but wrapped around provider's API.

Terraform - extensible and idempotent
- 100s of providers - terraform.io/providers

## 2. Extending Terraform - writing a provider
- Separate binaries - core and plugins
- Plugins are a gRPC server

"Creating a Terraform provider for just about anything" by Eddie Zanewski

## 3. Everything-as-Code - or get close

- Can we make a blog as code?
  - Medium provider - write and read only - not update in place or delete - blogs are immutable
  - Write blog posts in Markdown then run thru Terraform
  - Upload images separately, implement a resources

```
    resource "medium_image" "draft" {
	  file_path = "./images/draft.png"
	  content_type = "image/png"
	}
	
    resource "medium_post" "my-post" {
	  title = "Writing a Terraform Medium provider"
	  content = templateFile("./content.md", { image_url = medium_image.draft.url })
	  content_format = "markdown"
	  publish_status = "draft"
	}
```
- What happens when you have a non-CRUD API?
  - Maybe create a read-only provider ("data source" in Terraform)
  - Write you own client - write a wrapper around PHP to use an API
- Testing
  - Express functionality in acceptance tests
  - Examine interface changes with contract tests 
    - Medium provider - API does not allow you to read - had to go a public site, pull down, scrape JSON
	- Like any scrape, they may change things and break you
    - Better to test interface against a known resource (blog)
- What about other kinds of stuff-as-code? Make better workflows for yourself
  - Alerts as code (Datadog, PageDuty)
  - To-Do-as-Code (Google calendar, Todoist)
  - Life-as-Code (Meetup [community-as-code, tired to manually orchestrate user communities], Domino's pizza-as-code, Pokemon)
- "Can you offer translations?"
  - I don't have time, don't want to find native speakers
  - What if used Google translate API? Write a provider for it?
  - No jobs, Create-only
  - To translate and publish => chain the providers together?
  - To publish for Chinese readers on another site => translate provider + some other site publisher provider
- Whole point of DevOps is "doing better"-thru-Code
  - Empower somebody else to do better, through code

Seth Vargo - https://www.youtube.com/watch?v=HcmPi7-IVQo

- Got more complex over time
  - VMs
  - Chef, Ansible, etc. - works great for single servers
  - Now VMs, containers, bare metal - all kinds of combinations
  - But also everything-as-a-Service (all kinds of "aaS"'s)
- Doesn't even mention monitoring and alerting, security
- Thus: You need automation to manage this complexity
- Four pillars
  - Acquire - in the past, had to buy, ship, unpack, put in a rack
  - Provision - operations put on OS, users, S/W packages
  - Update - manage, upgrade, patch
  - Destroy - decommission, don't need any more
- Now: 
  - A - cloud, everything is a resource
  - P - confirmation management tools
- Now there are tools to do all of these things - puppet, ansible, k8s, chef, serverless, docker
- Why are there so many tools? So much complexity! But today all that tooling is necessary
  - Need a strategy for managing the complexity
  - So many times before the natural answer to managing complexity is code
  - Applications
- Answer: Codify
  - Capture process, routine, or algorithm in a textual format -> then processed by a tool
  - May be DECLARATIVE (desired state) or IMPERATIVE (steps to take)
  - E.g. Salt, puppet - all have languages to define and codify; for docker it's Dockerfile
  - IaC we have TF files processed by tool Terraform to bring about a desired result
  - CI/CD - .gitlab-ci.yaml, Jenkinsfile - CI/CD not driver thru UI
  - Security and policy complexity - need to manage and reason about security at scale - e.g. Vault has a code

Also from: https://medium.com/nationwide-technology/everything-as-code-lessons-from-applications-as-code-952e94603b1e

Q: Why is code a valuable tool for managing complexity? If saying we're doing EaC, we need to treat it like code. Have to do the same things to it.
- Development stories with acceptance criteria, QA walkthroughs and testing
- Repo and branching strategy
- Enforce our opinions on the code - static analysis, linting - enforce consistency. If you want adoption and want to collaborate, make sure it's consistent.
- Automated tests written for it
  - As soon as you make it code, you have the ability to test it
  - Anything from simple unit tests to complex integration tests
  - Can increment on it over time
- Collaborate on it 
  - merge requests/reviews, hard for one person to reason about it
- Separate concerns
  - Isolation layers, encapsulation
- Tests are run regularly and builds (if any) done in a CI pipeline
- Automated deployment to execution environment

Nothing new here - just the same stuff application developers have had for years
- So - if what's happening in operations world is what applications had 10 years ago, what do operations look like in 10 years?
  - Less operator intervention "my job is to make myself obsolete" 
    - operators don't fight fires, they inject them
	- we have autoscaling - do we have deep insights into exactly what we need to scale? proactive based on data and analysis instead of reactive as today?
  - Automated security scanning 
    - we have blackduck, AWS Inspector - rely on CVEs and CWEs
	- but can we add pen testing (like chaos monkeys)
	- automated security patching - is this something our culture really want? We have to comfortable with systems using automation.
  - More intelligent insights
    - What are biggest things in developer landscape? AI/ML
	- Q: Can we leverage AI/ML in operations workflow? Use it to analyze metrics to do more complex matching
	- Anomoly detection - machines are really good at this
	- So: when you have millions of log lines streaming, how do you know that one of those blips is an error, or a bad user experience, or an attacker?
	- Distributed tracing - what if you used AI/ML, see what are normal and abnormal paths through your systems
