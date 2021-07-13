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
