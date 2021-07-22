Event data and metrics data (MELT) - only events available in NRQL, not metrics. Unlike Insights - New Relic One combines accounts, can still show metrics.
Default event types you can use out of the box:
- PageView events - page in browser
- Transaction event - from APM agent, every time
- Synthetic check status and result
- Mobile session event - every time user accesses mobile device

Custom
- attributes - agents have an API - add user id, revenue, cart size, etc.
- events - add custom event type - agents can send custom events, or can use NR Event Insert API. Generally better to add custom attributes where you solve the problem.

Flex manager - "build your own integration" - comes bundled with infrastructure agent - add extra config files

Attribute dictionary - https://docs.newrelic.com/attribute-dictionary/

NRQL - no joins

SELECT function(attribute) <- REQUIRED
  FROM event <- case sensitive, REQUIRED
  WHERE attribute [comparison] AND|OR ...
  FACET attribute <- grouping to see results, like GROUP BY
  LIMIT number <- how many results at a time
  SINCE/UNTIL time <- control the time window, e.g SINCE 1 WEEK AGO or SINCE JUNE 15
  WITH TIMEZONE tz <- adjust the TZ
  COMPARE WITH time <- e.g. COMPARE WITH 1 HOUR AGO
  TIMESERIES time <- anything that comes in a number and see over time

Key/Value pair (KVP) - An event is a moment in time with multiple KVP
- 3 elements - EventType, Events, attributes
- EventType - collection of events (like a DB) with a shared origin; e.g. Transaction (powered by APM agents), PageView (powered by browser agents), SyntheticCheck (Synthetic monitors - event represents a monitor run)
- Event - single record in EventType table - each needs two keys, EventType + timestamp
- Attribute - a KVP stored in a particular event (e.g. duration=5000ms, browser=Chrome)

sleuth.io - SaaS - automatically tracking your Accelerate metrics - either thru plugin integrations or with an API. Have 60 code pipelines on random build tool. Can meet dev teams where they are.
- Accelerate metrics seem to be really important

NR powerschool-<district> eg dallasisd - same application name across all the nodes
- Events NR knows how to track
- Plus "PS" events sent via perf capture

Greg added perf capture NR instrumentation in 2017 - https://powerschoolgroup.atlassian.net/browse/INST-9301
Greg's doc on adding NR - https://powerschoolgroup.atlassian.net/wiki/spaces/~greg.stavenow@powerschool.com/pages/4538246/PowerSchool+New+Relic+JVM+Monitor
