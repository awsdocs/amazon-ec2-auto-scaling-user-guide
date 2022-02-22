# Configuring a notification target for lifecycle notifications<a name="configuring-lifecycle-hook-notifications"></a>

You can add lifecycle hooks to an Auto Scaling group to perform custom actions when an instance enters a wait state\. You can choose a target service to perform these actions depending on your preferred development approach\.

The first approach uses Amazon EventBridge to invoke a Lambda function that performs the action you want\. The second approach involves creating an Amazon Simple Notification Service \(Amazon SNS\) topic to which notifications are published\. Clients can subscribe to the SNS topic and receive published messages using a supported protocol\. The last approach involves using Amazon Simple Queue Service \(Amazon SQS\), a messaging system used by distributed applications to exchange messages through a polling model\.

As a best practice, we recommend that you use EventBridge\. The notifications sent to Amazon SNS and Amazon SQS contain the same information as the notifications that Amazon EC2 Auto Scaling sends to EventBridge\. Before EventBridge, the standard practice was to send a notification to SNS or SQS and integrate another service with SNS or SQS to perform programmatic actions\. Today, EventBridge gives you more options for which services you can target and makes it easier to handle events using serverless architecture\. 

The following procedures cover how to set up your notification target\.

Remember, if you have user data scripts or cloud\-init directives that configure your instances when they launch, you do not need to receive notification when the lifecycle action occurs\.

**Topics**
+ [Route notifications to Lambda using EventBridge](#cloudwatch-events-notification)
+ [Receive notifications using Amazon SNS](#sns-notifications)
+ [Receive notifications using Amazon SQS](#sqs-notifications)
+ [Notification message example for Amazon SNS and Amazon SQS](#notification-message-example)

**Important**  
The EventBridge rule, Lambda function, Amazon SNS topic, and Amazon SQS queue that you use with lifecycle hooks must always be in the same Region where you created your Auto Scaling group\.

## Route notifications to Lambda using EventBridge<a name="cloudwatch-events-notification"></a>

You can invoke a Lambda function when a lifecycle action occurs using EventBridge\. For information about the EventBridge events that are emitted when a lifecycle action occurs, see [Auto Scaling events](cloud-watch-events.md#cloudwatch-event-types)\. 

Before you begin, confirm that you have created a Lambda function\. For an introductory tutorial that shows you how to create a simple Lambda function that listens for launch events and writes them out to a CloudWatch Logs log, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\. 

**To use a Lambda function as the target of an EventBridge rule**

1. Create a Lambda function by using the [Lambda console](https://console.aws.amazon.com/lambda/home#/functions) and note its Amazon Resource Name \(ARN\)\. For example, `arn:aws:lambda:region:123456789012:function:my-function`\. You need the ARN to create an EventBridge target\.

   For more information, see [Getting started with Lambda](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html) in the *AWS Lambda Developer Guide*\.

1. Create an EventBridge rule that matches the lifecycle action using the following [put\-rule](https://docs.aws.amazon.com/cli/latest/reference/events/put-rule.html) command\.
**Note**  
When you use the AWS Management Console to create an event rule, the console automatically adds the IAM permissions necessary to grant EventBridge permission to call your Lambda function\. If you are creating an event rule using the AWS CLI, you need to grant this permission explicitly\. For help creating an event rule using the console, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\.

   ```
   aws events put-rule --name my-rule --event-pattern file://pattern.json --state ENABLED
   ```

   The following example shows the `pattern.json` for an instance launch lifecycle action\.

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-launch Lifecycle Action" ]
   }
   ```

   The following example shows the `pattern.json` for an instance terminate lifecycle action\.

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-terminate Lifecycle Action" ]
   }
   ```

1. Create a target that invokes your Lambda function when the lifecycle action occurs, using the following [put\-targets](https://docs.aws.amazon.com/cli/latest/reference/events/put-targets.html) command\.

   ```
   aws events put-targets --rule my-rule --targets Id=1,Arn=arn:aws:lambda:region:123456789012:function:my-function
   ```

1. Grant the rule permission to invoke your Lambda function using the following [add\-permission](https://docs.aws.amazon.com/cli/latest/reference/lambda/add-permission.html) command\. This command trusts the EventBridge service principal \(`events.amazonaws.com`\) and scopes permissions to the specified rule\.

   ```
   aws lambda add-permission --function-name my-function --statement-id my-unique-id \
     --action 'lambda:InvokeFunction' --principal events.amazonaws.com --source-arn arn:aws:events:region:123456789012:rule/my-rule
   ```

1. After you have followed these instructions, continue on to [Adding lifecycle hooks](adding-lifecycle-hooks.md) as a next step\.

## Receive notifications using Amazon SNS<a name="sns-notifications"></a>

You can use Amazon SNS to set up a notification target \(an SNS topic\) to receive notifications when a lifecycle action occurs\. Amazon SNS then sends the notifications to the subscribed recipients\. Until the subscription is confirmed, no notifications published to the topic are sent to the recipients\. 

**To set up notifications using Amazon SNS**

1. Create an Amazon SNS topic by using either the [Amazon SNS console](https://console.aws.amazon.com/sns/) or the following [create\-topic](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html) command\. Ensure that the topic is in the same Region as the Auto Scaling group that you're using\. For more information, see [Getting started with Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html) in the *Amazon Simple Notification Service Developer Guide*\. 

   ```
   aws sns create-topic --name my-sns-topic
   ```

1. Note the topic Amazon Resource Name \(ARN\), for example, `arn:aws:sns:region:123456789012:my-sns-topic`\. You need it to create the lifecycle hook\.

1. Create an IAM service role to give Amazon EC2 Auto Scaling access to your Amazon SNS notification target\.

    **To give Amazon EC2 Auto Scaling access to your SNS topic** 

   1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

   1. On the navigation pane, choose **Roles**, **Create role**\.

   1. Under **Select type of trusted entity**, choose **AWS service**\. 

   1. Under **Choose the service that will use this role**, choose **EC2 Auto Scaling** from the list\. 

   1. Under **Select your use case**, choose **EC2 Auto Scaling Notification Access**, and then choose **Next:Permissions**\. 

   1. Choose **Next:Tags**\. Optionally, you can add metadata to the role by attaching tags as key\-value pairs\. Then choose **Next:Review**\. 

   1. On the **Review** page, enter a name for the role \(for example, my\-notification\-role\), and choose **Create role**\. 

   1. On the **Roles** page, choose the role that you just created to open the **Summary** page\. Make a note of the **Role ARN**\. For example, `arn:aws:iam::123456789012:role/my-notification-role`\. You need it to create the lifecycle hook\.

1. After you have followed these instructions, continue on to [Add lifecycle hooks \(AWS CLI\)](adding-lifecycle-hooks.md#adding-lifecycle-hooks-aws-cli) as a next step\.

## Receive notifications using Amazon SQS<a name="sqs-notifications"></a>

You can use Amazon SQS to set up a notification target to receive messages when a lifecycle action occurs\. A queue consumer must then poll an SQS queue to act on these notifications\.

**Important**  
FIFO queues are not compatible with lifecycle hooks\.

**To set up notifications using Amazon SQS**

1. Create an Amazon SQS queue by using the [Amazon SQS console](https://console.aws.amazon.com/sqs/)\. Ensure that the queue is in the same Region as the Auto Scaling group that you're using\. For more information, see [Getting started with Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-getting-started.html) in the *Amazon Simple Queue Service Developer Guide*\. 

1. Note the queue ARN, for example, `arn:aws:sqs:region:123456789012:my-sqs-queue`\. You need it to create the lifecycle hook\.

1. Create an IAM service role to give Amazon EC2 Auto Scaling access to your Amazon SQS notification target\.

    **To give Amazon EC2 Auto Scaling access to your SQS queue** 

   1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

   1. On the navigation pane, choose **Roles**, **Create role**\.

   1. Under **Select type of trusted entity**, choose **AWS service**\. 

   1. Under **Choose the service that will use this role**, choose **EC2 Auto Scaling** from the list\. 

   1. Under **Select your use case**, choose **EC2 Auto Scaling Notification Access**, and then choose **Next:Permissions**\. 

   1. Choose **Next:Tags**\. Optionally, you can add metadata to the role by attaching tags as key\-value pairs\. Then choose **Next:Review**\. 

   1. On the **Review** page, enter a name for the role \(for example, my\-notification\-role\), and choose **Create role**\. 

   1. On the **Roles** page, choose the role you that just created to open the **Summary** page\. Make a note of the **Role ARN**\. For example, `arn:aws:iam::123456789012:role/my-notification-role`\. You need it to create the lifecycle hook\.

1. After you have followed these instructions, continue on to [Add lifecycle hooks \(AWS CLI\)](adding-lifecycle-hooks.md#adding-lifecycle-hooks-aws-cli) as a next step\.

## Notification message example for Amazon SNS and Amazon SQS<a name="notification-message-example"></a>

While the instance is in a wait state, a message is published to the Amazon SNS or Amazon SQS notification target\. The message includes the following information:
+ `LifecycleActionToken` — The lifecycle action token\.
+ `AccountId` — The Amazon Web Services account ID\.
+ `AutoScalingGroupName` — The name of the Auto Scaling group\.
+ `LifecycleHookName` — The name of the lifecycle hook\.
+ `EC2InstanceId` — The ID of the EC2 instance\.
+ `LifecycleTransition` — The lifecycle hook type\.
+ `NotificationMetadata` — The notification metadata\.

The following is a notification message example\.

```
Service: AWS Auto Scaling
Time: 2021-01-19T00:36:26.533Z
RequestId: 18b2ec17-3e9b-4c15-8024-ff2e8ce8786a
LifecycleActionToken: 71514b9d-6a40-4b26-8523-05e7ee35fa40
AccountId: 123456789012
AutoScalingGroupName: my-asg
LifecycleHookName: my-hook
EC2InstanceId: i-0598c7d356eba48d7
LifecycleTransition: autoscaling:EC2_INSTANCE_LAUNCHING
NotificationMetadata: hook message metadata
```

### Test notification message example<a name="test-notification-message-example"></a>

When you first add a lifecycle hook, a test notification message is published to the notification target\. The following is a test notification message example\.

```
Service: AWS Auto Scaling
Time: 2021-01-19T00:35:52.359Z
RequestId: 18b2ec17-3e9b-4c15-8024-ff2e8ce8786a
Event: autoscaling:TEST_NOTIFICATION
AccountId: 123456789012
AutoScalingGroupName: my-asg
AutoScalingGroupARN: arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:042cba90-ad2f-431c-9b4d-6d9055bcc9fb:autoScalingGroupName/my-asg
```

**Note**  
For examples of the events delivered from Amazon EC2 Auto Scaling to EventBridge, see [Auto Scaling events](cloud-watch-events.md#cloudwatch-event-types)\.