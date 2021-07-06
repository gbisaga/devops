CloudWatch events together with CodePipeline
- Auto-created rule tying in the pipeline with the source change, e.g. codepipeline-garyte-master-901422-rule
{
  "source": [
    "aws.codecommit"
  ],
  "detail-type": [
    "CodeCommit Repository State Change"
  ],
  "resources": [
    "arn:aws:codecommit:us-east-1:356371446905:garytest1"
  ],
  "detail": {
    "event": [
      "referenceCreated",
      "referenceUpdated"
    ],
    "referenceType": [
      "branch"
    ],
    "referenceName": [
      "master"
    ]
  }
}
- Can also create a scheduled rule, e.g. once a day invoke the pipeline
- Can also use source events
  - CodePipeline execution state change - FAILED, STARTED, SUCCEEDED, etc.
  - Target being SNS or Lambda (post to slack?)
{
  "source": [
    "aws.codepipeline"
  ],
  "detail-type": [
    "CodePipeline Stage Execution State Change"
  ],
  "detail": {
    "state": [
      "STARTED"
    ]
  }
}
- Shows a "sample event" - what you'll be getting e.g. in the Lambda
