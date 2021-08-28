Observability is:
- About being able to ask questions of software - what it's doing - and get answers
- Diff from metrics/logging/monitoring - being able to say "it's doing this, for this purpose, for this person"
- Because today it's not just one app server - hundreds of containers, microservices
- For people called dev, devops, product, support - are you in some way responsible for this product. Product manager may be closely watching performance. "I'm watching this metric, filter out this endpoint, on this platform..." Can pinpoint exactly what looking for, don't have to predict ahead of time.
- Not have to request access - anybody responsible for S/W can create and ask the questions
- Answering questions about your system using data
- Uses production data to inform what you develop: does use of a feature flag inform whether to do more dev? Is this thing we thought was an "edge case" really an edge case? Product managers hypothesize about these things, this lets us see what is really happening: is what they thought really being used the way they thought it would?
- Really empowering everybody - democratize the power. Often an engineering team with one repository of expertise. Want people to look it up themselves.

o11ycon - Observability con - Q: is our instrumentation good enough to make these decisions?
- About breaking down artificial silos - can we have internal PowerSchool groups like o11ycon, bringing together people from all up and down the tree?

Honeycomb.io "Observability guide"

Observability-Driven Development - you have a hypothesis about what you're doing, are we really doing it?

Observability over monitoring
- Can you answer any new questions, and not just the questions you're prepared for?
- Monitoring doesn't scale up to many kinds of services - e.g. serverless

Three/four pillars
- Logging - immutable, detailed timestamped record of discrete events that happened over time
- Tracing - series of causally related distributed events - relationships between entities
  - Getting data is difficult, manually instrument each layer
  - Open source making it easier
    - OpenTracing 2016 - make it easy to instrument data, but hard to integrate to binaries (k8s, DB) - open, vendor neutral API
	- OpenCensus 2018 - capturing and metrics - easier for binaries but harder for new software - complementary to OpenTracing
	- Merged 2019 - OpenTelemetry - combines the two
- Metrics - four golden signals - collected at regular intervals, timestamp/name/values/count of how many events
  - Latency - time to service requests
  - Traffic - requests/second
  - Error - error rate
  - Saturation - fullness of a service
- Event - discrete action at a particular time, e.g. user buying from a vending machine. Add detailed metadata, e.g. item category and cost
- Alerting? Don't we want to be notified? In an observability environment, want many less alerts.
  - Maybe want higher-level (more business)
  - Three categories of alerts
    - Page-worthy
    - Sub-critical - show up in work queues
    - More minor - keep on dashboards or queryable

From The Ops Show by CTO.ai https://www.youtube.com/watch?v=BXioGHTJOkc
- When something gets slow, everything gets slow - but why?
- Monitoring tools, you can look, but depend on common patterns
- Logs, it's great if you know what you're searching for
- Started to see light with Scuba at Facebook - https://research.fb.com/publications/scuba-diving-into-data-at-facebook/ 
  - one thing does well - query in real time on high cardinality data
  - what do all these problem cases have in common? Is it everything, or only stuff with these common cases
  - we engineers don't work with customer stuff enough - but the more we see the results of what we do, the better. 
- Next five years: Monitoring, logs, APM monitoring disappear
  - Categories born from scarcity - everything is so expensive to track
  - Observability is arbitrarily wide data blobs
  - KEY: You can go from observability to MLT, but not the other direction

Observability is supposed to be easy with monoliths, hard with microservices
- But not really true. A monolith has many pieces. Like SIS perf mon - a big help in finding internal problems
- We have that kind of monitoring also with yourkit

OpenTelemetry course https://www.youtube.com/watch?v=r8UvWSX3KA8
- Project + OpenTelemetry => standardized data => analysis tools for tracing, metrics, or both
- Tracing
  - Zipkin (tracing app) + Prometheus (metrics) = observability tool (NR One)
  - Contexts - two types
    - Scan context - metadata across invocations (trade id, span id, trace flags)
    - Correlation context - user defined properties (customer id, host name, region, etc.) - app specific - not required to be stored
  - Propagation - how to bundle up contexts and they transfer across services
    - Sent with W3C Trace Context headers
      https://docs.dapr.io/developing-applications/building-blocks/observability/w3c-tracing/w3c-tracing-overview/
  - Include tracing.js - separate module that instruments express calls
- Metrics
  - Prometheus monitoring app for metrics
  - Numeric representations of data over intervals of time
  - Since it's numbers, more compact storage, easier mathematical - allows aggregation over time
- What kind of issues can opentelemetry pick up?
  - Backend
	- Bad logic or user input
    - Poorly implemented backend/downstream calls - long response times or errors
    - Poorly performant code on a specific API
  - Frontend
	- Bad logic or user input
	- Poorly implemented JS, making UI slow despite
	- Locate geo-specific slowness (extra context)
	- Identify configuration changes, misconfigured DNS, noisy neighbors, etc.
- Movies/dashboard project
  - NodeTracerProvider - generates the spans - @opentelemetry/plugin-http is the plugin that traps http calls, and NodeTracerProvider picks up the plugin
  - Then we need an exporter to send the telemetry data somewhere. We create two exporters (console and zipkin) and add them to the provider NodeTracerProvider.
  - Note: a span can be a call, but it's really just literally a span of time, related to other spans of time. Can measure any individual part of service. Find out each piece of the system - middleware, handler, etc. The core plugins in the opentelemetry project are for the request level; but contrib plugins allow creating spans for specific frameworks.

Aggregating old results vs retention
- Peek season investigation - compare this year vs last year
- https://docs.newrelic.com/docs/telemetry-data-platform/manage-data/manage-data-retention/
- What is the required/appropriate event generation rate? Every msec is expensive, but 30 minutes is probably too slow. Probably want some guidance around these kinds of questions.

New Relic One AWS observability workshop https://newrelic.awsworkshop.io/

"Why observability and monitoring are different" https://www.youtube.com/watch?v=2ivl8Hzp7QQ&ab_channel=HasgeekTV

Three examples:
- If you have two servers, 55%, is this good?
- How do you know the #s you're monitoring for are correct?
- Support ticket on ELB - a year after deployed, finding thru access logs node1 always has higher load than node3

Q: Can we find out before things go sideways?
Standard A: Monitor. Problem: you monitor what you know, react after it's failed. 
- It's known failures: cannot monitor/alert for unknown-unknowns. There is nothing that says "monitor everything"
- Also don't look across distributed systems: monitoring generally a single CPU, single process.
- Properties we don't usually look at within these distributed systems: accuracy, latency, correctness, consistency.
  - Example: we have eventual consistency. How eventual is eventual?
- Monitoring also tell you from now: the hooks and levers have to be in the software. You need to have installed the agent, or monitor, or alert.

Software, by default, is opaque: so you need obversation pre-built into the system.
- Tracing allows you to see what happened
- Logs are great, but you can't normally connect them from different services
- The need
  - Debugging
  - Pattern detection - I see a pattern, everytime it goes to service B, it degrades; but not when it goes to C
- Design for debuggability
  - Operational (external) failures should be handled; programmatic ones should be debugged
  - The more software designed for debuggability, less you need to debug it; and the more you can leverage services around it. (C service has no problem, so we can look at B more.)
- Observability is not failure-centric, unlike monitoring
  - Just because it's observable, doesn't mean anybody is (or needs to) observe it
  - But you can, for any number of reasons (debugging, pattern detection)

Stability - Apply Control theory
- You can't build 1 MB and 1 TB system the same way - all systems have a bounded nature
- Phone > LB > backend system... put in a sensor and you have a closed loop system
- Example: Route53 weighted balancing
- Another example: Request > Envoy > HTTP payment service then Hystrix for feedback
- A simple feedback system: try HTTP call, if fail make a retry. How long backoff? How many times? Etc.
  - More complex, circuit breaking - realize a certain endpoint is failing to some critical level -> do transactions asynchronously
 to that DB.
  - Feedback helps you build more reliable software
  - TCP flow control good example of feedback
  - TCP congestion control takes it one more level - deals with network failures proactively. 
    - "Slow start" - Exponential growth of trusting the other side
	- No network is stable - every time there's a packet drop, you fall back to slow start
  - Caching systems are feedback loop also - LRU, etc.
  - DDoS protection, WAF
  - Load balancer - receiver says (somehow) I can't take any more traffic
  - Progressive streaming - no matter how bad your network is, it works. This is because bandwidth is measured for every segment; then bandwidth is reduced if problems.

Programming language/environment selection
- Support for tools for testing (mocking, API testing, functional UI testing), linting, CI/CD, observability
- Use that meets team standards (OO, functional, etc.)

Observability specific - ThoughtWorks https://www.thoughtworks.com/en-us/insights/podcasts/technology-podcasts/oberservability-monitoring
- Language environment note
- Coverage (e.g. Fitness functions)
- Proactive, not reactive
- Main idea is that it's distributed
- Use open standard - need to decide if OpenTracing or now OpenTelemetry - or can we use an implementation that will allow this to be changed in the future?
- Include trace info in logs, if they are not sent together

Thoughts on creating an observable system
- Another one - how do you know what SQL is being run?
- Log collection - different sources
- Generate logs with significant application events
- Custom metrics significant to the application? That way you can correlate it after the fact with problem times. But, maybe this can also be done by generating the log messages, and then creating custom metrics from them. (Practice exam question)

- CI/CD and cost first - features later
