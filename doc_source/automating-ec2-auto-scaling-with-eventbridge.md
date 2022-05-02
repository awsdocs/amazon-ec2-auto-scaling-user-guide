# Use EventBridge to handle Auto Scaling events<a name="automating-ec2-auto-scaling-with-eventbridge"></a>

Amazon EventBridge, formerly called CloudWatch Events, helps you set up event\-driven rules that monitor resources and initiate target actions that use other AWS services\.

Events from Amazon EC2 Auto Scaling are delivered to EventBridge in near\-real time\. You can establish EventBridge rules that trigger programmatic actions and notifications in response to a variety of these events\. For example, while instances are in the process of launching or terminating, you can invoke an AWS Lambda function to perform a preconfigured task\.

In addition to invoking Lambda functions, EventBridge supports other types of targets and actions, such as relaying events to Amazon Kinesis streams, activating AWS Step Functions state machines, and invoking the AWS Systems Manager run command\. For information about supported targets, see [Amazon EventBridge targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-targets.html) in the *Amazon EventBridge User Guide*\.

For more information about EventBridge, see [Getting started with Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-getting-set-up.html) in the *Amazon EventBridge User Guide*\.

**Note**  
You can also find additional documentation for using EventBridge in the *Amazon EC2 Auto Scaling User Guide*\.  
For a step\-by\-step tutorial that shows you how to use lifecycle hooks to put instances in a wait state before invoking a Lambda function that processes EventBridge events, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\.
To create EventBridge rules that detect events associated with a warm pool, see [Create EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md)\.
To create an Amazon SNS topic and EventBridge rule that notifies you whenever a checkpoint is reached during an instance refresh, see [Create EventBridge rules for instance refresh events](monitor-events-eventbridge-sns.md)\.

**Topics**
+ [Auto Scaling group events](#auto-scaling-group-event-types)
+ [Warm pool events](#warm-pool-events)
+ [Use Auto Scaling events to invoke an AWS Lambda function](#handle-events-using-lambda)

## Auto Scaling group events<a name="auto-scaling-group-event-types"></a>

The following are example events from Amazon EC2 Auto Scaling\. Events are emitted on a best\-effort basis\. 

**Topics**
+ [EC2 Instance\-launch Lifecycle Action](#launch-lifecycle-action)
+ [EC2 Instance Launch Successful](#launch-successful)
+ [EC2 Instance Launch Unsuccessful](#launch-unsuccessful)
+ [EC2 Instance\-terminate Lifecycle Action](#terminate-lifecycle-action)
+ [EC2 Instance Terminate Successful](#terminate-successful)
+ [EC2 Instance Terminate Unsuccessful](#terminate-unsuccessful)
+ [EC2 Auto Scaling Instance Refresh Checkpoint Reached](#instance-refresh-checkpoint-reached)
+ [EC2 Auto Scaling Instance Refresh Started](#instance-refresh-started)
+ [EC2 Auto Scaling Instance Refresh Succeeded](#instance-refresh-succeeded)
+ [EC2 Auto Scaling Instance Refresh Failed](#instance-refresh-failed)
+ [EC2 Auto Scaling Instance Refresh Cancelled](#instance-refresh-cancelled)

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

### EC2 Auto Scaling Instance Refresh Started<a name="instance-refresh-started"></a>

Amazon EC2 Auto Scaling emits events when the status of an instance refresh changes to `InProgress`\. 

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Started",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```

### EC2 Auto Scaling Instance Refresh Succeeded<a name="instance-refresh-succeeded"></a>

Amazon EC2 Auto Scaling emits events when the status of an instance refresh changes to `Succeeded`\. 

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Succeeded",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```

### EC2 Auto Scaling Instance Refresh Failed<a name="instance-refresh-failed"></a>

Amazon EC2 Auto Scaling emits events when the status of an instance refresh changes to `Failed`\. 

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Failed",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```

### EC2 Auto Scaling Instance Refresh Cancelled<a name="instance-refresh-cancelled"></a>

Amazon EC2 Auto Scaling emits events when the status of an instance refresh changes to `Cancelled`\. 

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Cancelled",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```

## Warm pool events<a name="warm-pool-events"></a>

For examples of the events that are delivered to EventBridge when you add a warm pool to your Auto Scaling group, see [Warm pool events](warm-pools-eventbridge-events.md#warm-pool-event-types)\.

## Use Auto Scaling events to invoke an AWS Lambda function<a name="handle-events-using-lambda"></a>

AWS Lambda is a compute service that you can use to run code without provisioning or managing servers\. You package your code and upload it to AWS Lambda as a *Lambda function*\. AWS Lambda then runs the function when the function is invoked\. A function can be invoked manually by you, automatically in response to events, or in response to requests from applications or services\. 

To get started with Lambda, use the following procedure\. This section shows you how to create a Lambda function and an EventBridge rule causing all instance launch and terminate events to be logged in Amazon CloudWatch Logs\. 

### Create a Lambda function<a name="create-lambda-function"></a>

Use the following procedure to create a Lambda function using the **hello\-world** blueprint to serve as the target for events\.

**To create a Lambda function**

1. Open the AWS Lambda console at [https://console\.aws\.amazon\.com/lambda/](https://console.aws.amazon.com/lambda/)\.

1. If you are new to Lambda, you see a welcome page; choose **Get Started Now**; otherwise, choose **Create a Lambda function**\.

1. On the **Select blueprint** page, enter `hello-world` for **Filter**, and then select the **hello\-world** blueprint\.

1. On the **Configure triggers** page, choose **Next**\.

1. On the **Configure function** page, do the following:

   1. Enter a name and description for the Lambda function\.

   1. Edit the code for the Lambda function\. For example, the following code logs the event\.

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

   1. For **Service provider**, choose **Amazon Web Services**\.

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

### See also<a name="eventbridge-see-also"></a>

For a step\-by\-step tutorial that shows you how to create an EventBridge rule that invokes a Lambda function on Amazon EC2 or Amazon EC2 Auto Scaling API calls, see [Tutorial: Log AWS API calls using EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-log-api-call.html) in the *Amazon EventBridge User Guide*\.