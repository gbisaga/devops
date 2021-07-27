Key takeaway - (1) deploy when you want (2) release when you are ready

LaunchDarkly - https://www.youtube.com/watch?v=0evo68QmcJ4
- Test in production - a bunch of people doing it
- Three reasons why
  - Validate direction - if actual customers don't use it, just making assumptions
  - Feedback and inspiration - customers have ideas, not depending on vision of PMs
  - Incremental delivery - this is a cultural thing - need to overcome the idea of "this must be perfect". In fact, we don't really know things are perfect now, just that it worked for most of what we tried.
- Turn on features for certain accounts
- Triad - testing in production is all about the engineering+product+design (iron triangle?)

How to test in production
- Everything is feature flagged, forever
- Have actual control over product features
  - Customer "I want this" - Us "You've got it, right now" (and vice versa)
- "Secret shopping" bring in outsiders to judge what's there
- Test in production for real, with production load, in larger context (including all their data, and all their customizations)
- Experimentation - which color increased the number of clicks?

Major questions:
- Are feature flags tech debt?
- How about different combinations that have never been tested together? Are flags at multiple levels? Flag A requires flag B?
- How do we do our own testing to make sure it works with flags on and off?

Steps:
0. Flagging during the planning phase
  - Agree on what needs flagging ("does pagination decrease page load time?" "Does this button help?")
    - Dev flag backend logic - switch back to old backend behavior (go back to default before we implemented pagination)
	- PMs dynamically configure page values - try 50 vs 100 results on a page
	- Designs may want to A/B test
  - Problem: predicting ahead of time is hard. So flag it all.

1. All dev is feature flagged
2. Define rollout strategy - very important, exciting. 
  - You decide your phases, who gets them. What is timing, how many customers?
  - Define criteria to go to 100% (GA)
  - Define your kill switch (?)

3. Deploy to production and go home, let PM's test with whoever they want
4. Begin testing with your rollout strategy
5. Build scalable feedback look
- LaunchDarkly is full stack - not just UI elements: search algorithms, experiments on page load time
- To build a feedback loop: Create an experiment: associate a metric plus a flag
  - Flag is five different variations - where to place page numbers on the page, have a default of no pagination. 5 variations, turn on each for 20% of customers.
  - Metric is conversion rate for page
  - Run an experiment, PMs turn on flag in various combinations, look for best conversion rate with acceptable confidence interval
  - Then PM turns on 100%

Feature flag has several components:
- Whether on/off
- What the roles are - who can do what

How feature flagging lets you test in production
- Single workflow with dev team from concept to launch
- Build for incremental and faster value delivery
- Test out ideas and variations at scale, quick feedback
- Control access and features for peace of mind

Good, bad and ugly https://www.youtube.com/watch?v=r7VI5x2XKXw
- Amazing, but can do really bad things - case studies
- Knight Capital, a financial services firm - dropped by feature flags and deployments
  - https://dougseven.com/2014/04/17/knightmare-a-devops-cautionary-tale/
  - PowerPeg and SMARS - PowerPeg code hadn't been used for 8 years, but decided to develop SMARS
  - SMARS tested fine in QA - deployed as a dark release... except one server still had the old code
  - Reused the PowerPeg feature toggle; so when they toggled it on, one server was running old software
  - Went to back out, so they rolled back... but without turning off the feature flag. Now ALL servers were using the old PowerPeg feature
  - Then they turned off the toggle, and things were good... but lost $245M in 45 minutes
- HP printer firmware
  - Were spending 5% of the time working on new features - feedback cycle was 8 weeks "Did you work on this thing 8 weeks ago?"
  - Originally dev branches for each printer type - tons of merges between branches
  - Moved to trunk based development with feature flags - on startup, it reads the printer's model number
  - Moved to 2 days to see that a feature works through simulation and emulation

Rules around feature flags:
- Rule 1: NEVER re-use a feature toggle.
- Q: Why is dead code - a feature not used in 8 years - still in the codebase? Somebody might turn it back on! (Eliminate with trunk based development - just make a release that deletes it, and roll back)
- Rule 2: Feature toggles should have a short lifespan - with one exception, up to weeks.
  - So: you can use the feature flags as a list of code to review
- Rule 3: Name your toggles well - clearly. Various options:
  - If `EnablePowerPeg` they wouldn't have re-used it, because it was obviously wrong. 
  - Maybe name after a jira ticket (`PAY-123`) - this can work if you have a description also
  - Probably something like `MakeMoneyFastFeature`
- Rule 4: Architecture matters - reduce the number of places that have to know about toggles
  - You could put `if (_toggles.PowerPeg.Enabled)` throughout your code - but no better than #IFDEF everywhere
  - Better to branch by abstraction - kind of replaces source control branching 
    - define an interface with different implementations, then dependency injection - good for branch on startup
	- define a decorator - centralizing the toggle check into one place
	- in the front end you can wrap the toggle
  - So the key idea: reduce the number of places that have to know about toggles - better for testing
- Rule 5: Monitor toggles
  - Need to know if toggle is on or off, and how often
  - If it's always on (or always off), delete it

hwhap-20210715-release-

Four categories of times when a toggle is checked (axis of flexibility)
- Compile - not common, very invasive
- Startup time toggle, read once - change value, you restart to get it
- Periodic - fetches in the background periodically
- Activity - extra overhead to fetch from e.g. LaunchDarkly - reality is, Activity = Periodic with a very short period

Four usages of toggles
- Release - often for a subset of users, e.g. everybody in beta testing group, or 10%
- Experiment toggle - similar to release, but more short term "should we use blue or red button?"
- Ops - allowed to be lived for a long time - e.g. feature that might have a performance impact
- Permission - for a particular user or group - this is probably the wrong tool for this purpose, because permissions should always be done using an authorization system. Only maybe feature toggles for a very short period, before authorization system is available

Ops                                     Y           Y       -> these are usually system-wide, for everybody, usually almost immediate
Experiment                  Y           Y           Y       -> the riskier the experiment, the farther to the right
Release         ?           Y           Y
             Compile     Startup     Periodic    Activity

Complaints
- Makes testing harder - argues it doesn't except for a short period of time
- Adds complexity - everything adds complexity, maybe we can abstract it; and even if it adds complexity to the code, less complexity to the process

Just talked with Flaten about this, and we both concur: yes to squashing, as long as it will cause no problems when we merge back into matterhorn later. Assuming there won't be merge conflicts when we go back into matterhorn later, the only reasons I can imagine to not squash are:
- we want history, but the p4 branching model was essentially the same as if we squash; and
- cherry picking
- multiple commits for the same jira ticket
 
Lyft - no external service - called "runtime"
- github repo - client https://github.com/lyft/goruntime
- uses locally managed flags - stored in a different repo
- Q: How do you merge the code rollouts with the feature flag rollouts?

Constant reading 
- ok with disk-based
- but maybe problems with incomplete cached items
- also hard if you're deploying docker - how do you update the disk files?

Front end and back end
- can get back from back end (def backend features with frontend consequences)
- but web frontend is its own docker container
- frontend not doing SPAs from CloudFront - server side rendering for react apps - react JSX can be server side
- even before server side rendering, everything is a node layer

Naming convention?
- Different types - boolean and percentages (mostly for percentage of rollout)
- Data directory > Rides directory > { Default (irrespective of env), Dev, Prod, Staging }
- Three categories: Flags, Kill switches, Parameters
- File name is the name of the flag "enable_somefeature.percentage"

Testing combinations?
- If flags part of repo, you always test combinations
- What if they interact with each other? General answer: don't

Inline vs injection, etc.
- Lots of solutions across different services
- Working on one - abstracting if/else block with conditionally rendering components
- Frontend - sometimes easier - 2 Angular <div>'s with if/else

Experiment - data science, experiment 

Monorepo - pull the whole thing?
