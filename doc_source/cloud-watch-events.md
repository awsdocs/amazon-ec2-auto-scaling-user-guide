# Using Amazon EC2 Auto Scaling with EventBridge<a name="cloud-watch-events"></a>

Amazon EventBridge, formerly called CloudWatch Events, helps you set up event\-driven rules that monitor resources and initiate target actions that use other AWS services\.

Events from Amazon EC2 Auto Scaling are delivered to EventBridge in near real time\. You can establish EventBridge rules that trigger programmatic actions and notifications in response to a variety of these events\. For example, while instances are in the process of launching or terminating, you can trigger an AWS Lambda function to perform a preconfigured task\. Or, you can trigger notifications to an Amazon SNS topic to monitor the progress of an instance refresh and to perform validations at specific checkpoints\.

In addition to invoking Lambda functions and notifying Amazon SNS topics, EventBridge supports other types of targets and actions, such as relaying events to Amazon Kinesis streams, activating AWS Step Functions state machines, and invoking the AWS Systems Manager run command\. For information about supported targets, see [Amazon EventBridge targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-targets.html) in the *Amazon EventBridge User Guide*\.

For more information about EventBridge, see [Getting started with Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-getting-set-up.html) in the *Amazon EventBridge User Guide*\. Note that you can also create rules that trigger on Amazon EC2 Auto Scaling API calls\. For more information, see [Creating an EventBridge rule that triggers on an AWS API call using AWS CloudTrail](https://docs.aws.amazon.com/eventbridge/latest/userguide/create-eventbridge-cloudtrail-rule.html) in the *Amazon EventBridge User Guide*\. 

**Topics**
+ [Auto Scaling events](#cloudwatch-event-types)
+ [Using AWS Lambda to handle events](#handle-events-using-lambda)
+ [Creating EventBridge rules for instance refresh events](#monitor-events-eventbridge-sns)

## Auto Scaling events<a name="cloudwatch-event-types"></a>

The following are example events from Amazon EC2 Auto Scaling\. Events are emitted on a best effort basis\. 

For examples of the events delivered from Amazon EC2 Auto Scaling to EventBridge when you use a warm pool, see [Warm pool events](warm-pools-eventbridge-events.md#warm-pool-event-types)\. 

For an example of the event for a Spot Instance interruption, see [Spot Instance interruption notices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#spot-instance-termination-notices) in the *Amazon EC2 User Guide for Linux Instances*\. 

**Topics**
+ [EC2 Instance\-launch Lifecycle Action](#launch-lifecycle-action)
+ [EC2 Instance Launch Successful](#launch-successful)
+ [EC2 Instance Launch Unsuccessful](#launch-unsuccessful)
+ [EC2 Instance\-terminate Lifecycle Action](#terminate-lifecycle-action)
+ [EC2 Instance Terminate Successful](#terminate-successful)
+ [EC2 Instance Terminate Unsuccessful](#terminate-unsuccessful)
+ [EC2 Auto Scaling Instance Refresh Checkpoint Reached](#instance-refresh-checkpoint-reached)

### EC2 Instance\-launch Lifecycle Action<a name="launch-lifecycle-action"></a>

Amazon EC2 Auto Scaling moved an instance to a `Pending:Wait` state due to a lifecycle hook\. 

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance-launch Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": { 
    "LifecycleActionToken": "87654321-4321-4321-4321-210987654321", 
    "AutoScalingGroupName": "my-asg", 
    "LifecycleHookName": "my-lifecycle-hook", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info"
  } 
}
```

### EC2 Instance Launch Successful<a name="launch-successful"></a>

Amazon EC2 Auto Scaling successfully launched an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Launch Successful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
      "auto-scaling-group-arn",
      "instance-arn"
  ],
  "detail": {
      "StatusCode": "InProgress",
      "Description": "Launching a new EC2 instance: i-12345678",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

### EC2 Instance Launch Unsuccessful<a name="launch-unsuccessful"></a>

Amazon EC2 Auto Scaling failed to launch an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Launch Unsuccessful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn",
    "instance-arn"
  ],
  "detail": {
      "StatusCode": "Failed",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "message-text",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

### EC2 Instance\-terminate Lifecycle Action<a name="terminate-lifecycle-action"></a>

Amazon EC2 Auto Scaling moved an instance to a `Terminating:Wait` state due to a lifecycle hook\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance-terminate Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": { 
    "LifecycleActionToken":"87654321-4321-4321-4321-210987654321", 
    "AutoScalingGroupName":"my-asg", 
    "LifecycleHookName":"my-lifecycle-hook", 
    "EC2InstanceId":"i-1234567890abcdef0", 
    "LifecycleTransition":"autoscaling:EC2_INSTANCE_TERMINATING", 
    "NotificationMetadata":"additional-info"
  } 
}
```

### EC2 Instance Terminate Successful<a name="terminate-successful"></a>

Amazon EC2 Auto Scaling successfully terminated an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Terminate Successful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn",
    "instance-arn"
  ],
  "detail": {
      "StatusCode": "InProgress",
      "Description": "Terminating EC2 instance: i-12345678",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

### EC2 Instance Terminate Unsuccessful<a name="terminate-unsuccessful"></a>

Amazon EC2 Auto Scaling failed to terminate an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Terminate Unsuccessful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn",
    "instance-arn"
  ],
  "detail": {
      "StatusCode": "Failed",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "message-text",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

### EC2 Auto Scaling Instance Refresh Checkpoint Reached<a name="instance-refresh-checkpoint-reached"></a>

During an instance refresh, Amazon EC2 Auto Scaling emits events when the number of instances that have been replaced reaches the percentage threshold defined for the checkpoint\. 

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Checkpoint Reached",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "ab00cf8f-9126-4f3c-8010-dbb8cad6fb86",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "CheckpointPercentage": "50",
      "CheckpointDelay": "300"
  }
}
```

## Using AWS Lambda to handle events<a name="handle-events-using-lambda"></a>

AWS Lambda is a compute service that you can use to run code without provisioning or managing servers\. You package your code and upload it to AWS Lambda as a *Lambda function*\. AWS Lambda then runs the function when the function is invoked\. A function can be invoked manually by you, automatically in response to events, or in response to requests from applications or services\. 

To help you get started with Lambda, follow the procedure below\. This section shows you how to create a Lambda function and an EventBridge rule causing all instance launch and terminate events to be logged in Amazon CloudWatch Logs\. 

For a step\-by\-step tutorial that shows you how to create a Lambda function to perform lifecycle actions, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\. This tutorial covers how to get started with lifecycle hooks\. With lifecycle hooks, you can use Lambda to perform tasks on instances before they are put into service or before they are terminated\.

### Create a Lambda function<a name="create-lambda-function"></a>

Use the following procedure to create a Lambda function using the **hello\-world** blueprint to serve as the target for events\.

**To create a Lambda function**

1. Open the AWS Lambda console at [https://console\.aws\.amazon\.com/lambda/](https://console.aws.amazon.com/lambda/)\.

1. If you are new to Lambda, you see a welcome page; choose **Get Started Now**; otherwise, choose **Create a Lambda function**\.

1. On the **Select blueprint** page, enter `hello-world` for **Filter**, and then select the **hello\-world** blueprint\.

1. On the **Configure triggers** page, choose **Next**\.

1. On the **Configure function** page, do the following:

   1. Enter a name and description for the Lambda function\.

   1. Edit the code for the Lambda function\. For example, the following code simply logs the event\.

      ```
      console.log('Loading function');
      
      exports.handler = function(event, context) {
          console.log("AutoScalingEvent()");
          console.log("Event data:\n" + JSON.stringify(event, null, 4));
          context.succeed("...");
      };
      ```

   1. For **Role**, choose **Choose an existing role**\. For **Existing role**, select your basic execution role\. Otherwise, create a basic execution role\.

   1. \(Optional\) For **Advanced settings**, make any changes that you need\.

   1. Choose **Next**\.

1. On the **Review** page, choose **Create function**\.

### Route events to your Lambda function<a name="create-rule"></a>

Create a rule that matches selected events and routes them to your Lambda function to take action\. 

**To create a rule that routes events to your Lambda function**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, under **Events**, choose **Rules**\.

1. Choose **Create rule**\.

1. Enter a name and description for the rule\.

1. For **Define pattern**, do the following:

   1. Choose **Event Pattern**\.

   1. For **Event matching pattern**, choose **Pre\-defined by service**\.
**Tip**  
You can also create a rule that uses a custom event pattern to detect and act upon only a subset of Amazon EC2 Auto Scaling events\. This subset can be based on specific fields that Amazon EC2 Auto Scaling includes in its events\. For more information, see [Event patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/filtering-examples-structure.html) in the *Amazon EventBridge User Guide*\. For an example event pattern, see [Step 3: Create an EventBridge rule](tutorial-lifecycle-hook-lambda.md#lambda-create-rule) in the tutorial for lifecycle hooks\.

   1. For **Service provider**, choose **AWS**\.

   1. For **Service Name**, choose **Auto Scaling**\.

   1. For **Event type**, choose **Instance Launch and Terminate**\.

   1. To capture all successful and unsuccessful instance launch and terminate events, choose **Any instance event**\.

   1. By default, the rule matches any Auto Scaling group in the Region\. To make the rule match a specific Auto Scaling group, choose **Specific group name\(s\)** and select one or more Auto Scaling groups\.

1. For **Select event bus**, choose **AWS default event bus**\. When an AWS service in your account emits an event, it always goes to your account's default event bus\. 

1. For **Target**, choose **Lambda function**\.

1. For **Function**, select the Lambda function that you created\.

1. Choose **Create**\.

To test your rule, change the size of your Auto Scaling group\. If you used the example code for your Lambda function, it logs the event to CloudWatch Logs\.

**To test your rule**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. On the **Details** tab, choose **Edit** from the right side of the page\.

1. Change the value of **Desired capacity**, and then choose **Update**\.

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation pane, choose **Logs**\.

1. Select the log group for your Lambda function \(for example, **/aws/lambda/***my\-function*\)\.

1. Select a log stream to view the event data\. The data is displayed, similar to the following:  
![\[Viewing event data for Amazon EC2 Auto Scaling in CloudWatch Logs.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/auto_scaling_event_data.png)

## Creating EventBridge rules for instance refresh events<a name="monitor-events-eventbridge-sns"></a>

This section shows you how to create a rule that notifies you whenever a checkpoint is reached during an instance refresh\. The procedure for setting up email notifications through Amazon SNS is included\. To use Amazon SNS to send email notifications, you must first create a *topic* and then subscribe your email addresses to the topic\.

### Create an Amazon SNS topic<a name="eventbridge-sns-create-topic"></a>

 An SNS topic is a logical access point, a communication channel that your Auto Scaling group uses to send the notifications\. You create a topic by specifying a name for your topic\.

When you create a topic name, the name must meet the following requirements:
+ Between 1 and 256 characters long
+ Contain uppercase and lowercase ASCII letters, numbers, underscores, or hyphens 

For more information, see [Creating an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.

### Subscribe to the Amazon SNS topic<a name="eventbridge-sns-subscribe-topic"></a>

To receive the notifications that your Auto Scaling group sends to the topic, you must subscribe an endpoint to the topic\. In this procedure, for **Endpoint**, specify the email address where you want to receive the notifications from Amazon EC2 Auto Scaling\.

For more information, see [Subscribing to an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-subscribe-endpoint-to-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.

### Confirm your Amazon SNS subscription<a name="eventbridge-sns-confirm-subscription"></a>

Amazon SNS sends a confirmation email to the email address you specified in the previous step\.

Make sure that you open the email from AWS Notifications and choose the link to confirm the subscription before you continue with the next step\.

You will receive an acknowledgment message from AWS\. Amazon SNS is now configured to receive notifications and send the notification as an email to the email address that you specified\.

### Route events to your Amazon SNS topic<a name="eventbridge-sns-create-rule"></a>

Create a rule that matches selected events and routes them to your Amazon SNS topic to notify subscribed email addresses\.

**To create a rule that routes events to your Amazon SNS topic**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, under **Events**, choose **Rules**\.

1. In the **Rules** section, choose **Create rule**\.

1. Enter a name and description for the rule\.

1. For **Define pattern**, do the following:

   1. Choose **Event Pattern**\.

   1. For **Event matching pattern**, choose **Pre\-defined by service**\.

   1. For **Service provider**, choose **AWS**\.

   1. For **Service Name**, choose **Auto Scaling**\.

   1. For **Event type**, choose **Instance Refresh**\.

   1. By default, the rule matches any Auto Scaling group in the Region\. To make the rule match a specific Auto Scaling group, choose **Specific group name\(s\)** and select one or more Auto Scaling groups\.

1. For **Select event bus**, choose **AWS default event bus**\. When an AWS service in your account emits an event, it always goes to your account's default event bus\. 

1. For **Target**, choose **SNS topic**\.

1. For **Topic**, select the Amazon SNS topic that you created\.

1. For **Configure input**, choose the input for the email notification\. 

1. Choose **Create**\.