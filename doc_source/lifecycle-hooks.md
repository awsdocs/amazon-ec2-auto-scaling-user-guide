# Amazon EC2 Auto Scaling Lifecycle Hooks<a name="lifecycle-hooks"></a>

*Lifecycle hooks* enable you to perform custom actions by pausing instances as an Auto Scaling group launches or terminates them\. For example, while your newly launched instance is paused, you could install or configure software on it\.

Each Auto Scaling group can have multiple lifecycle hooks\. However, there is a limit on the number of hooks per Auto Scaling group\. For more information, see [Auto Scaling Limits](as-account-limits.md)\.

**Topics**
+ [How Lifecycle Hooks Work](#lifecycle-hooks-overview)
+ [Considerations When Using Lifecycle Hooks](#lifecycle-hook-considerations)
+ [Prepare for Notifications](#preparing-for-notification)
+ [Add Lifecycle Hooks](#adding-lifecycle-hooks)
+ [Complete the Lifecycle Hook](#completing-lifecycle-hooks)
+ [Test the Notification](#testing-hook-notifications)

## How Lifecycle Hooks Work<a name="lifecycle-hooks-overview"></a>

After you add lifecycle hooks to your Auto Scaling group, they work as follows:

1. Responds to scale out events by launching instances and scale in events by terminating instances\.

1. Puts the instance into a wait state \(`Pending:Wait` or `Terminating:Wait`\)\. The instance is paused until either you continue or the timeout period ends\.

1. You can perform a custom action using one or more of the following options:
   + Define a CloudWatch Events target to invoke a Lambda function when a lifecycle action occurs\. The Lambda function is invoked when Amazon EC2 Auto Scaling submits an event for a lifecycle action to CloudWatch Events\. The event contains information about the instance that is launching or terminating, and a token that you can use to control the lifecycle action\.
   + Define a notification target for the lifecycle hook\. Amazon EC2 Auto Scaling sends a message to the notification target\. The message contains information about the instance that is launching or terminating, and a token that you can use to control the lifecycle action\.
   + Create a script that runs on the instance as the instance starts\. The script can control the lifecycle action using the ID of the instance on which it runs\.

1. By default, the instance remains in a wait state for one hour, and then the Auto Scaling group continues the launch or terminate process \(`Pending:Proceed` or `Terminating:Proceed`\)\. If you need more time, you can restart the timeout period by recording a heartbeat\. If you finish before the timeout period ends, you can complete the lifecycle action, which continues the launch or termination process\.

The following illustration shows the transitions between instance states in this process:

![\[The lifecycle of instances using lifecycle hooks.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/lifecycle_hooks.png)

For more information about the complete lifecycle of instances in an Auto Scaling group, see [Auto Scaling Lifecycle](AutoScalingGroupLifecycle.md)\.

## Considerations When Using Lifecycle Hooks<a name="lifecycle-hook-considerations"></a>

Adding lifecycle hooks to your Auto Scaling group gives you greater control over how instances launch and terminate\. Here are some things to consider when adding a lifecycle hook to your Auto Scaling group, to help ensure that the group continues to perform as expected\.

**Topics**
+ [Keeping Instances in a Wait State](#lifecycle-hook-wait-state)
+ [Cooldowns and Custom Actions](#lifecycle-hook-cooldowns)
+ [Health Check Grace Period](#lifecycle-hook-health-check-grace-period)
+ [Lifecycle Action Result](#lifecycle-hook-results)
+ [Spot Instances](#lifecycle-hook-spot)

### Keeping Instances in a Wait State<a name="lifecycle-hook-wait-state"></a>

Instances can remain in a wait state for a finite period of time\. The default is 1 hour \(3600 seconds\)\. You can adjust this time in the following ways:
+ Set the heartbeat timeout for the lifecycle hook when you create the lifecycle hook\. With the [put\-lifecycle\-hook](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command, use the `--heartbeat-timeout` parameter\. With the PutLifecycleHook operation, use the `HeartbeatTimeout` parameter\.
+ Continue to the next state if you finish before the timeout period ends, using the [complete\-lifecycle\-action](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command or the CompleteLifecycleAction operation\.
+ Restart the timeout period by recording a heartbeat, using the [record\-lifecycle\-action\-heartbeat](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) command or the RecordLifecycleActionHeartbeat operation\. This increments the heartbeat timeout by the timeout value specified when you created the lifecycle hook\. For example, if the timeout value is 1 hour, and you call this command after 30 minutes, the instance remains in a wait state for an additional hour, or a total of 90 minutes\.

The maximum amount of time that you can keep an instance in a wait state is 48 hours or 100 times the heartbeat timeout, whichever is smaller\.

### Cooldowns and Custom Actions<a name="lifecycle-hook-cooldowns"></a>

When an Auto Scaling group launches or terminates an instance due to a simple scaling policy, a [cooldown](Cooldown.md) takes effect\. The cooldown period helps ensure that the Auto Scaling group does not launch or terminate more instances than needed\.

Consider an Auto Scaling group with a lifecycle hook that supports a custom action at instance launch\. When the application experiences an increase in demand, the group launches instances to add capacity\. Because there is a lifecycle hook, the instance is put into the `Pending:Wait` state, which means that it is not available to handle traffic yet\. When the instance enters the wait state, scaling actions due to simple scaling policies are suspended\. When the instance enter the `InService` state, the cooldown period starts\. When the cooldown period expires, any suspended scaling actions resume\.

### Health Check Grace Period<a name="lifecycle-hook-health-check-grace-period"></a>

If you add a lifecycle hook to perform actions as your instances launch, the health check grace period does not start until you complete the lifecycle hook and the instance enters the `InService` state\.

### Lifecycle Action Result<a name="lifecycle-hook-results"></a>

At the conclusion of a lifecycle hook, the result is either `ABANDON` or `CONTINUE`\.

If the instance is launching, `CONTINUE` indicates that your actions were successful, and that the instance can be put into service\. Otherwise, `ABANDON` indicates that your custom actions were unsuccessful, and that the instance can be terminated\.

If the instance is terminating, both `ABANDON` and `CONTINUE` allow the instance to terminate\. However, `ABANDON` stops any remaining actions, such as other lifecycle hooks, while `CONTINUE` allows any other lifecycle hooks to complete\.

### Spot Instances<a name="lifecycle-hook-spot"></a>

You can use lifecycle hooks with Spot Instances\. However, a lifecycle hook does not prevent an instance from terminating due to a change in the Spot Price, which can happen at any time\. In addition, when a Spot Instance terminates, you must still complete the lifecycle action \(using the complete\-lifecycle\-action command or the CompleteLifecycleAction operation\)\.

## Prepare for Notifications<a name="preparing-for-notification"></a>

You can optionally configure notifications when the instance enters a wait state, which enables you to perform a custom action\. You can use Amazon CloudWatch Events, Amazon SNS, or Amazon SQS to receive the notifications\. Choose whichever option you prefer\.

Alternatively, if you have a script that configures your instances when they launch, you do not need to receive notification when the lifecycle action occurs\. If you are not doing so already, update your script to retrieve the instance ID of the instance from the instance metadata\. For more information, see [Retrieving Instance Metadata](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-retrieval)\.

**Topics**
+ [Receive Notification Using CloudWatch Events](#cloudwatch-events-notification)
+ [Receive Notification Using Amazon SNS](#sns-notifications)
+ [Receive Notification Using Amazon SQS](#sqs-notifications)

### Receive Notification Using CloudWatch Events<a name="cloudwatch-events-notification"></a>

You can use CloudWatch Events to set up a target to invoke a Lambda function when a lifecycle action occurs\.

**To set up notifications using CloudWatch Events**

1. Create a Lambda function using the steps in [Create a Lambda Function](cloud-watch-events.md#create-lambda-function) and note its Amazon Resource Name \(ARN\)\. For example, `arn:aws:lambda:us-west-2:123456789012:function:my-function`\.

1. Create a CloudWatch Events rule that matches the lifecycle action using the following [put\-rule](http://docs.aws.amazon.com/cli/latest/reference/events/put-rule.html) command:

   ```
   aws events put-rule --name my-rule --event-pattern file://pattern.json --state ENABLED
   ```

   The `pattern.json` for an instance launch lifecycle action is:

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-launch Lifecycle Action" ]
   }
   ```

   The `pattern.json` for an instance terminate lifecycle action is:

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-terminate Lifecycle Action" ]
   }
   ```

1. Grant the rule permission to invoke your Lambda function using the following [add\-permission](http://docs.aws.amazon.com/cli/latest/reference/lambda/add-permission.html) command\. This command trusts the CloudWatch Events service principal \(`events.amazonaws.com`\) and scopes permissions to the specified rule\.

   ```
   aws lambda add-permission --function-name LogScheduledEvent --statement-id my-scheduled-event --action 'lambda:InvokeFunction' --principal events.amazonaws.com --source-arn arn:aws:events:us-east-1:123456789012:rule/my-scheduled-rule
   ```

1. Create a target that invokes your Lambda function when the lifecycle action occurs, using the following [put\-targets](http://docs.aws.amazon.com/cli/latest/reference/events/put-targets.html) command:

   ```
   aws events put-targets --rule my-rule --targets Id=1,Arn=arn:aws:lambda:us-west-2:123456789012:function:my-function
   ```

1. When the Auto Scaling group responds to a scale\-out or scale\-in event, it puts the instance in a wait state\. While the instance is in a wait state, the Lambda function is invoked\. For more information about the event data, see [Auto Scaling Events](cloud-watch-events.md#cloudwatch-event-types)\.

### Receive Notification Using Amazon SNS<a name="sns-notifications"></a>

You can use Amazon SNS to set up a notification target to receive notifications when a lifecycle action occurs\.

**To set up notifications using Amazon SNS**

1. Create the target using Amazon SNS\. For more information, see [Create a Topic](http://docs.aws.amazon.com/sns/latest/dg/CreateTopic.html) in the *Amazon Simple Notification Service Developer Guide*\. Note the ARN of the target \(for example, `arn:aws:sns:us-west-2:123456789012:my-sns-topic`\)\.

1. Create an IAM role to grant Amazon EC2 Auto Scaling permissions to access your notification target, using the steps in [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\. When prompted to select a role type, select **AWS Service Roles**, **AutoScaling Notification Access**\. Note the ARN of the role\. For example, `arn:aws:iam::123456789012:role/my-notification-role`\.

1. When the Auto Scaling group responds to a scale\-out or scale in event, it puts the instance in a wait state\. While the instance is in a wait state, a message is published to the notification target\. The message includes the following event data:
   + **LifecycleActionToken** — The lifecycle action token\.
   + **AccountId** — The AWS account ID\.
   + **AutoScalingGroupName** — The name of the Auto Scaling group\.
   + **LifecycleHookName** — The name of the lifecycle hook\.
   + **EC2InstanceId** — The ID of the EC2 instance\.
   + **LifecycleTransition** — The lifecycle hook type\.

   For example:

   ```
   Service: AWS Auto Scaling
   Time: 2016-09-30T20:42:11.305Z
   RequestId: 18b2ec17-3e9b-4c15-8024-ff2e8ce8786a
   LifecycleActionToken: 71514b9d-6a40-4b26-8523-05e7ee35fa40
   AccountId: 123456789012
   AutoScalingGroupName: my-asg
   LifecycleHookName: my-hook
   EC2InstanceId: i-0598c7d356eba48d7
   LifecycleTransition: autoscaling:EC2_INSTANCE_LAUNCHING
   NotificationMetadata: null
   ```

### Receive Notification Using Amazon SQS<a name="sqs-notifications"></a>

You can use Amazon SQS to set up a notification target to receive notifications when a lifecycle action occurs\.

**Important**  
FIFO queues are not compatible with lifecycle hooks\.

**To set up notifications using Amazon SQS**

1. Create the target using Amazon SQS\. For more information, see [Getting Started with Amazon SQS](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-getting-started.html) in the *Amazon Simple Queue Service Developer Guide*\. Note the ARN of the target\.

1. Create an IAM role to grant Amazon EC2 Auto Scaling permission to access your notification target, using the steps in [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\. When prompted to select a role type, select **AWS Service Roles**, **AutoScaling Notification Access**\. Note the ARN of the role\. For example, `arn:aws:iam::123456789012:role/my-notification-role`\.

1. When the Auto Scaling group responds to a scale\-out or scale\-in event, it puts the instance in a wait state\. While the instance is in a wait state, a message is published to the notification target\.

## Add Lifecycle Hooks<a name="adding-lifecycle-hooks"></a>

You can create lifecycle hooks using the [put\-lifecycle\-hook](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command\.

To perform an action on scale out, use the following command:

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-hook --auto-scaling-group-name my-asg --lifecycle-transition autoscaling:EC2_INSTANCE_LAUNCHING
```

To perform an action on scale in, use the following command instead:

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-hook --auto-scaling-group-name my-asg --lifecycle-transition autoscaling:EC2_INSTANCE_TERMINATING
```

\(Optional\) To receive notifications using Amazon SNS or Amazon SQS, there are additional options\. For example, add the following options to specify an SNS topic as the notification target:

```
--notification-target-arn arn:aws:sns:us-west-2:123456789012:my-sns-topic --role-arn arn:aws:iam::123456789012:role/my-notification-role
```

The topic receives a test notification with the following key\-value pair:

```
"Event": "autoscaling:TEST_NOTIFICATION"
```

## Complete the Lifecycle Hook<a name="completing-lifecycle-hooks"></a>

When an Auto Scaling group responds to a scale out or scale in event, it puts the instance in a wait state and sends any notifications\. It continues the launch or terminate process after you complete the lifecycle hook\.

**To complete a lifecycle hook**

1. While the instance is in a wait state, you can perform a custom action\. For more information, see [Prepare for Notifications](#preparing-for-notification)\.

1. If you need more time to complete the custom action, use the [record\-lifecycle\-action\-heartbeat](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) command to restart the timeout period and keep the instance in a wait state\. You can specify the lifecycle action token you received in the previous step, as shown in the following command:

   ```
   aws autoscaling record-lifecycle-action-heartbeat --lifecycle-action-token bcd2f1b8-9a78-44d3-8a7a-4dd07d7cf635 --lifecycle-hook-name my-launch-hook --auto-scaling-group-name my-asg
   ```

   Alternatively, you can specify the ID of the instance you retrieved in the previous step, as shown in the following command:

   ```
   aws autoscaling record-lifecycle-action-heartbeat --instance-id i-1a2b3c4d --lifecycle-hook-name my-launch-hook --auto-scaling-group-name my-asg
   ```

1. If you finish the custom action before the timeout period ends, use the [complete\-lifecycle\-action](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command so that the Auto Scaling group can continue launching or terminating the instance\. You can specify the lifecycle action token, as shown in the following command:

   ```
   aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE --lifecycle-action-token bcd2f1b8-9a78-44d3-8a7a-4dd07d7cf635 --lifecycle-hook-name my-launch-hook --auto-scaling-group-name my-asg
   ```

   Alternatively, you can specify the ID of the instance, as shown in the following command:

   ```
   aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE --instance-id i-1a2b3c4d --lifecycle-hook-name my-launch-hook --auto-scaling-group-name my-asg
   ```

## Test the Notification<a name="testing-hook-notifications"></a>

To generate a notification for a launch event, update the Auto Scaling group by increasing the desired capacity of the Auto Scaling group by 1\. You receive a notification within a few minutes after instance launch\.

**To change the desired capacity using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Details** tab, choose **Edit**\.

1. For **Desired**, increase the current value by 1\. If this value exceeds **Max**, you must also increase the value of **Max** by 1\.

1. Choose **Save**\.

1. After a few minutes, you'll receive notification for the event\. If you do not need the additional instance that you launched for this test, you can decrease **Desired** by 1\. After a few minutes, you'll receive notification for the event\.