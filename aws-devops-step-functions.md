KEY IDEA: Step functions used to orchestrate complex work flows - build all kinds of automations

Use cases - review https://aws.amazon.com/step-functions/use-cases/
- Used when you have complex work flow, want to make more visual and intuitive, more granular
- Example: Transcode media files S3 upload -> Lambda -> Trigger step function -> Invoke others
  - You could use Lambda for all of this, but hard to develop (also might run into execution time limits?)
- Example: Sequencing several AWS Batch jobs - transitions between jobs
  - Lambda doesn't work because not only do you pay for it the whole time, might time out
  - Step functions can take a year to run to completion
- Example: Publish events from a 

State machine definition
- JSON document with states, results, transitions
- Get a visual view of the state machine - shows visual path thru the machine
- Transitions can SUCCEED or FAIL overall state machine execution
- Tracks each execution - good for troubleshooting - retrace the event history

CloudWatch integration
- Events integration
  - Example: Source = Step Functions, Event = status change ERROR -> target = send a message to slack or call Lambda
  - Example: Source = Scheduled every 1 day -> target = Execute step function state machine
- Logs integration
  - As of Feb 2020, there is logs integration
  - Can use logs to generate metrics through subscriptions
