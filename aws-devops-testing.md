Types of testing
- Linting - like spell checking
- Unit testing - smallest unit possible, typically just does one single thing e.g. upper casing a word
- Functional testing - make sure section of code
- Integration testing - usually systems integrating with each other
- End-to-end testing - go thru like a normal user would
- Load testing - answer question: can my application scale?
- Chaos testing - hard to achieve, add chaos to the environment, make sure it keeps working e.g. deleting EC2 instances - tests DR

Ways to run
- Local - linting, unit, sometimes functional
- Build server - linting, unit, functional, integration, end-to-end, load testing
- Chaos testing - usually external servers, not in the pipeline; the more in the pipeline, the more guarantee it works in production
