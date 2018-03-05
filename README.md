# Code Pipeline Slack Notifier

This small Lambda app will post updates to a Slack channel for pipeline events generated by AWS Code Pipeline.

## Prerequisites

* AWS Account using Code Pipeline. Otherwise there's not much point to this. :)
* Slack account, and an **incoming webhook** URL you can use to post to it. This will be something like `https://hooks.slack.com/services/....`
* An AWS user with sufficient privileges to deploy the included CloudFormation stack

If build / deploying from source:

* An S3 bucket that you can use as a location of your build artifact. You should have write privileges to this
from your AWS user.
* Locally: Node, and an at least relatively recent version of the AWS CLI

## Setup

### Using pre-packaged CloudFormation package

This only works if you are deployed to the us-east-1 region. If you aren't, use the section _From source_ below.

#### From a terminal

Run the following from a terminal, substituting `YOUR-INCOMING-WEBHOOK-URL` for the Slack URL described above:

```bash
    $ aws cloudformation deploy --template-file ./prebuilt-templates/packaged-template.yaml --stack-name cp-slack-notifier --parameter-overrides SlackUrl=YOUR-INCOMING-WEBHOOK-URL --capabilities CAPABILITY_IAM
    
```

1. Wait for CloudFormation to do its thing

1. Trigger a CodePipeline build, and wait for the magic!

#### From the Web Console

1. Go to the Cloud Formation Web Console at https://console.aws.amazon.com/cloudformation

1. Click _Create Stack_

1. In the _Choose a template_ section click _Upload a template to Amazon S3_ and select `prebuilt-templates/packaged-template.yaml`

1. Enter a name for the new CloudFormation stack, and the Slack URL you configured above, and click _Next_.

1. Leave everything as is (or change if necessary for permissions) and click _Next_.

1. Acknowledge the capabilities section, and then click _Create Change Set_

1. When that is complete, click _Execute_ .

### From source

1. Run the following from a terminal, substituting `YOUR_S3_BUCKET` for the S3 bucket described above, and `YOUR-INCOMING-WEBHOOK-URL` for the Slack URL described above:

    ```bash
    $ npm install

    $ npm run dist

    $ aws cloudformation package --template-file sam.yml --s3-bucket YOUR_S3_BUCKET --output-template-file target/packaged-template.yaml

    $ aws cloudformation deploy --template-file ./target/packaged-template.yaml --stack-name cp-slack-notifier --parameter-overrides SlackUrl=YOUR-INCOMING-WEBHOOK-URL --capabilities CAPABILITY_IAM
    ```

1. Wait for CloudFormation to do its thing

1. Trigger a CodePipeline build, and wait for the magic!

## Possible extensions / modifications

1. To change which types of events you want to be notified about you can specify a filter within the
Cloudwatch Event rule. For instance, to only notify on failed runs of the pipeline, update the `EventPattern` section
of the `CloudPipelineEvent` in the `sam.yml` file as follows:

    ```yaml
      EventPattern:
        source:
        - aws.codepipeline
        detail-type:
        - CodePipeline Pipeline Execution State Change
        detail:
          state:
          - FAILED
    ```

1. Similarly, to only notify for certain pipelines you can use the `pipeline` attribute of the `detail` element.
For more details of event filtering, see the documentation at http://docs.aws.amazon.com/codepipeline/latest/userguide/detect-state-changes-cloudwatch-events.html

1. You can also listen for different, more granular, events within CodePipeline. This is also detailed in the link above.

1. You can perform filtering in code, in the Lambda function, but that will mean your Lambda function
is getting triggered more often, which can lead to increased costs.

1. Another nice extension would be different types of message for different kinds of events, or posting to different channels for different pipelines. Either of these are possible by modifying the message POSTed to Slack. See https://api.slack.com/docs/messages for more details

## Teardown

To teardown the notifier delete the stack through the Web Console, or use the incantation below:

```bash
$ aws cloudformation delete-stack --stack-name cp-slack-notifier
```

-----
Copyright 2017, Symphonia LLC
