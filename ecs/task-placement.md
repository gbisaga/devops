### Task placement and constraints
- Task launchtype EC2 ONLY, ECS must determine where to placement
  - Constraints of CPU, memory, ports
  - Hints
- When service scales in, ECS determine which tasks to terminate
- Process
  - Identify CPU, memory, port requirements
  - Task placement constraints
  - Strategoies
  - Pick an Instance

### Strategies - applied to a service
- Strategy is best effort
- Binpack 
  - lowest cost
  - least available amount of CPU or memory (pack in as many tasks as possible)
  - choose CPU or memory
- Random
- Spread
  - Best availability
  - Evenly spread on the value - instanceid, AZ
- You can mix them for a single service

### Task placement constraints
- distinctInstance - only one per EC2
- memberOf - use Cluster Query Language - e.g. only on t2.* instance types
