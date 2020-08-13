# Configuring notifications for Amazon EC2 Auto Scaling lifecycle hooks<a name="configuring-lifecycle-hook-notifications"></a>

You can add a lifecycle hook to an Auto Scaling group that triggers a notification when an instance enters a wait state\. You can configure these notifications for a variety of reasons, for example, to invoke a Lambda function or to receive email notification so that you can perform a custom action\. This topic describes how to configure notifications using Amazon EventBridge, Amazon SNS, and Amazon SQS\. Choose whichever option you prefer\. Alternatively, if you have a script that configures your instances when they launch, you do not need to receive notification when the lifecycle action occurs\.

**Important**  
AWS resources for notifications must always be created in the same AWS Region where you create your lifecycle hook\. For example, if you configure notifications using Amazon SNS, the Amazon SNS topic must reside in the same Region as your lifecycle hook\. 

**Topics**
+ [Route notifications to Lambda using EventBridge](#cloudwatch-events-notification)
+ [Receive notification using Amazon SNS](#sns-notifications)
+ [Receive notification using Amazon SQS](#sqs-notifications)

## Route notifications to Lambda using EventBridge<a name="cloudwatch-events-notification"></a>

You can use EventBridge to set up a target to invoke a Lambda function when a lifecycle action occurs\.

**To set up notifications using EventBridge**

1. Create a Lambda function using the steps in [Create a Lambda function](cloud-watch-events.md#create-lambda-function) and note its Amazon Resource Name \(ARN\)\. For example, `arn:aws:lambda:region:123456789012:function:my-function`\.

1. Create an EventBridge rule that matches the lifecycle action using the following [https://docs.aws.amazon.com/cli/latest/reference/events/put-rule.html](https://docs.aws.amazon.com/cli/latest/reference/events/put-rule.html) command\.

   ```
   aws events put-rule --name my-rule --event-pattern file://pattern.json --state ENABLED
   ```

   The following examples shows the `pattern.json` for an instance launch lifecycle action\.

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-launch Lifecycle Action" ]
   }
   ```

   The following examples shows the `pattern.json` for an instance terminate lifecycle action\.

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-terminate Lifecycle Action" ]
   }
   ```

1. Grant the rule permission to invoke your Lambda function using the following [https://docs.aws.amazon.com/cli/latest/reference/lambda/add-permission.html](https://docs.aws.amazon.com/cli/latest/reference/lambda/add-permission.html) command\. This command trusts the EventBridge service principal \(`events.amazonaws.com`\) and scopes permissions to the specified rule\.

   ```
   aws lambda add-permission --function-name LogScheduledEvent --statement-id my-scheduled-event \
     --action 'lambda:InvokeFunction' --principal events.amazonaws.com --source-arn arn:aws:events:region:123456789012:rule/my-scheduled-rule
   ```

1. Create a target that invokes your Lambda function when the lifecycle action occurs, using the following [https://docs.aws.amazon.com/cli/latest/reference/events/put-targets.html](https://docs.aws.amazon.com/cli/latest/reference/events/put-targets.html) command\.

   ```
   aws events put-targets --rule my-rule --targets Id=1,Arn=arn:aws:lambda:region:123456789012:function:my-function
   ```

1. After you have followed these instructions, continue on to [Add lifecycle hooks ](lifecycle-hooks.md#adding-lifecycle-hooks) as a next step\.

When the Auto Scaling group responds to a scale\-out or scale\-in event, it puts the instance in a wait state\. While the instance is in a wait state, the Lambda function is invoked\. For more information about the event data, see [Auto Scaling events](cloud-watch-events.md#cloudwatch-event-types)\.

## Receive notification using Amazon SNS<a name="sns-notifications"></a>

You can use Amazon SNS to set up a notification target to receive notifications when a lifecycle action occurs\. 

**To set up notifications using Amazon SNS**

1. Create an Amazon SNS topic using the following [https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html) command\. For more information, see [Create a topic](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html#CreateTopic) in the *Amazon Simple Notification Service Developer Guide*\. 

   ```
   aws sns create-topic --name my-sns-topic
   ```

   Note the ARN of the target \(for example, `arn:aws:sns:region:123456789012:my-sns-topic`\)\.

1. Create a service role \(or *assume* role\) for Amazon EC2 Auto Scaling to which you can grant permission to access your notification target\.

   **To create an IAM role and allow Amazon EC2 Auto Scaling to assume it**

   1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

   1. In the navigation pane, choose **Roles**, **Create new role**\.

   1. Under **Select type of trusted entity**, choose **AWS service**\. 

   1. Under **Choose the service that will use this role**, choose **EC2 Auto Scaling** from the list\. 

   1. Under **Select your use case**, choose **EC2 Auto Scaling Notification Access**, and then choose **Next:Permissions**\. 

   1. Choose **Next:Tags**\. Optionally, you can add metadata to the role by attaching tags as key\-value pairs\. Then choose **Next:Review**\. 

   1. On the **Review** page, enter a name for the role \(for example, my\-notification\-role\), and choose **Create role**\. 

   1. On the **Roles** page, choose the role that you just created to open the **Summary** page\. Make a note of the **Role ARN**\. For example, `arn:aws:iam::123456789012:role/my-notification-role`\. You will specify the role ARN when you create the lifecycle hook in the next procedure\. 

1. After you have followed these instructions, continue on to [Add lifecycle hooks \(AWS CLI\)](lifecycle-hooks.md#adding-lifecycle-hooks-aws-cli) as a next step\.

When the Auto Scaling group responds to a scale\-out or scale\-in event, it puts the instance in a wait state\. While the instance is in a wait state, a message is published to the notification target\. The message includes the following event data:
+ `LifecycleActionToken` — The lifecycle action token\.
+ `AccountId` — The AWS account ID\.
+ `AutoScalingGroupName` — The name of the Auto Scaling group\.
+ `LifecycleHookName` — The name of the lifecycle hook\.
+ `EC2InstanceId` — The ID of the EC2 instance\.
+ `LifecycleTransition` — The lifecycle hook type\.

The following is a notification message example\.

```
Service: AWS Auto Scaling
Time: 2019-04-30T20:42:11.305Z
RequestId: 18b2ec17-3e9b-4c15-8024-ff2e8ce8786a
LifecycleActionToken: 71514b9d-6a40-4b26-8523-05e7ee35fa40
AccountId: 123456789012
AutoScalingGroupName: my-asg
LifecycleHookName: my-hook
EC2InstanceId: i-0598c7d356eba48d7
LifecycleTransition: autoscaling:EC2_INSTANCE_LAUNCHING
NotificationMetadata: null
```

## Receive notification using Amazon SQS<a name="sqs-notifications"></a>

You can use Amazon SQS to set up a notification target to receive notifications when a lifecycle action occurs\. 

**Important**  
FIFO queues are not compatible with lifecycle hooks\.

**To set up notifications using Amazon SQS**

1. Create the target using Amazon SQS\. For more information, see [Getting started with Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-getting-started.html) in the *Amazon Simple Queue Service Developer Guide*\. Note the ARN of the target \(for example, `arn:aws:sqs:region:123456789012:my-sqs-queue`\)\.

1. Create a service role \(or *assume* role\) for Amazon EC2 Auto Scaling to which you can grant permission to access your notification target\.

   **To create an IAM role and allow Amazon EC2 Auto Scaling to assume it**

   1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

   1. In the navigation pane, choose **Roles**, **Create new role**\.

   1. Under **Select type of trusted entity**, choose **AWS service**\. 

   1. Under **Choose the service that will use this role**, choose **EC2 Auto Scaling** from the list\. 

   1. Under **Select your use case**, choose **EC2 Auto Scaling Notification Access**, and then choose **Next:Permissions**\. 

   1. Choose **Next:Tags**\. Optionally, you can add metadata to the role by attaching tags as key\-value pairs\. Then choose **Next:Review**\. 

   1. On the **Review** page, enter a name for the role \(for example, my\-notification\-role\), and choose **Create role**\. 

   1. On the **Roles** page, choose the role you that just created to open the **Summary** page\. Make a note of the **Role ARN**\. For example, `arn:aws:iam::123456789012:role/my-notification-role`\. You will specify the role ARN when you create the lifecycle hook in the next procedure\. 

1. After you have followed these instructions, continue on to [Add lifecycle hooks \(AWS CLI\)](lifecycle-hooks.md#adding-lifecycle-hooks-aws-cli) as a next step\.

When the Auto Scaling group responds to a scale\-out or scale\-in event, it puts the instance in a wait state\. While the instance is in a wait state, a message is published to the notification target\.