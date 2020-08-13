# Amazon EC2 Auto Scaling lifecycle hooks<a name="lifecycle-hooks"></a>

Lifecycle hooks enable you to perform custom actions by *pausing* instances as an Auto Scaling group launches or terminates them\. When an instance is paused, it remains in a wait state either until you complete the lifecycle action using the complete\-lifecycle\-action command or the `CompleteLifecycleAction` operation, or until the timeout period ends \(one hour by default\)\.

For example, let's say that your newly launched instance completes its startup sequence and a lifecycle hook pauses the instance\. While the instance is in a wait state, you can install or configure software on it, making sure that your instance is fully ready before it starts receiving traffic\. For another example of the use of lifecycle hooks, let's say that when a scale\-in event occurs, the terminating instance is first deregistered from the load balancer \(if the Auto Scaling group is being used with Elastic Load Balancing\)\. Then, a lifecycle hook pauses the instance before it is terminated\. While the instance is in the wait state, you can, for example, connect to the instance and download logs or other data before the instance is fully terminated\. 

Each Auto Scaling group can have multiple lifecycle hooks\. However, there is a limit on the number of hooks per Auto Scaling group\. For more information, see [Amazon EC2 Auto Scaling service quotas](as-account-limits.md)\. 

**Topics**
+ [How lifecycle hooks work](#lifecycle-hooks-overview)
+ [Considerations](#lifecycle-hook-considerations)
+ [Prepare for notifications](#preparing-for-notification)
+ [Add lifecycle hooks](#adding-lifecycle-hooks)
+ [Complete a lifecycle hook custom action](#completing-lifecycle-hooks)
+ [Test the notification](#testing-hook-notifications)
+ [Configuring notifications for Amazon EC2 Auto Scaling lifecycle hooks](configuring-lifecycle-hook-notifications.md)

## How lifecycle hooks work<a name="lifecycle-hooks-overview"></a>

After you add lifecycle hooks to your Auto Scaling group, they work as follows:

1. The Auto Scaling group responds to scale\-out events by launching instances and scale\-in events by terminating instances\.

1. The lifecycle hook puts the instance into a wait state \(`Pending:Wait` or `Terminating:Wait`\)\. The instance is paused until you continue or the timeout period ends\.

1. You can perform a custom action using one or more of the following options:
   + Define an EventBridge target to invoke a Lambda function when a lifecycle action occurs\. The Lambda function is invoked when Amazon EC2 Auto Scaling submits an event for a lifecycle action to EventBridge\. The event contains information about the instance that is launching or terminating, and a token that you can use to control the lifecycle action\.
   + Define a notification target for the lifecycle hook\. Amazon EC2 Auto Scaling sends a message to the notification target\. The message contains information about the instance that is launching or terminating, and a token that you can use to control the lifecycle action\.
   + Create a script that runs on the instance as the instance starts\. The script can control the lifecycle action using the ID of the instance on which it runs\.

1. By default, the instance remains in a wait state for one hour, and then the Auto Scaling group continues the launch or terminate process \(`Pending:Proceed` or `Terminating:Proceed`\)\. If you need more time, you can restart the timeout period by recording a heartbeat\. If you finish before the timeout period ends, you can complete the lifecycle action, which continues the launch or termination process\.

The following illustration shows the transitions between instance states in this process:

![\[The lifecycle of instances using lifecycle hooks.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/lifecycle_hooks.png)

For more information about the complete lifecycle of instances in an Auto Scaling group, see [Auto Scaling instance lifecycle](AutoScalingGroupLifecycle.md)\.

## Considerations<a name="lifecycle-hook-considerations"></a>

Adding lifecycle hooks to your Auto Scaling group gives you greater control over how instances launch and terminate\. The following are things to consider when adding a lifecycle hook to your Auto Scaling group, to help ensure that the group continues to perform as expected\.

### Keeping instances in a wait state<a name="lifecycle-hook-wait-state"></a>

Instances can remain in a wait state for a finite period of time\. The default is one hour \(3600 seconds\)\. You can adjust this time in the following ways:
+ Set the heartbeat timeout for the lifecycle hook when you create the lifecycle hook\. With the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command, use the `--heartbeat-timeout` parameter\. With the `PutLifecycleHook` operation, use the `HeartbeatTimeout` parameter\.
+ Continue to the next state if you finish before the timeout period ends, using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command or the `CompleteLifecycleAction` operation\.
+ Postpone the end of the timeout period by recording a heartbeat, using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) command or the `RecordLifecycleActionHeartbeat` operation\. This extends the timeout period by the timeout value specified when you created the lifecycle hook\. For example, if the timeout value is one hour, and you call this command after 30 minutes, the instance remains in a wait state for an additional hour, or a total of 90 minutes\.

The maximum amount of time that you can keep an instance in a wait state is 48 hours or 100 times the heartbeat timeout, whichever is smaller\.

### Cooldown periods for simple scaling<a name="lifecycle-hook-cooldowns"></a>

When an Auto Scaling group launches or terminates an instance due to a simple scaling policy, a cooldown period takes effect\. The cooldown period helps ensure that the Auto Scaling group does not launch or terminate more instances than needed before the effects of previous simple scaling activities are visible\. When a lifecycle action occurs, and an instance enters the wait state, scaling activities due to simple scaling policies are paused\. When the lifecycle hook execution finishes, the cooldown period starts\. If you set a long interval for the cooldown period, it will take more time for scaling to resume\. For more information, see [Scaling cooldowns for Amazon EC2 Auto Scaling](Cooldown.md)\.

### Health check grace period<a name="lifecycle-hook-health-check-grace-period"></a>

If you add a lifecycle hook, the health check grace period does not start until the lifecycle hook actions complete and the instance enters the `InService` state\.

### Spot Instances<a name="lifecycle-hook-spot"></a>

You can use lifecycle hooks with Spot Instances\. However, a lifecycle hook does not prevent an instance from terminating in the event that capacity is no longer available\. In addition, when a Spot Instance terminates, you must still complete the lifecycle action \(using the complete\-lifecycle\-action command or the `CompleteLifecycleAction` operation\)\.

## Prepare for notifications<a name="preparing-for-notification"></a>

You can configure notifications for when an instance enters a wait state\. You can use Amazon EventBridge, Amazon SNS, or Amazon SQS to receive the notifications\. For more information, see [Configuring lifecycle hook notifications](configuring-lifecycle-hook-notifications.md)\.

Alternatively, if you have a script that configures your instances when they launch, you do not need to receive notification when the lifecycle action occurs\. If you are not doing so already, update your script to retrieve the instance ID of the instance from the instance metadata\. For more information, see [Retrieving instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-retrieval) in the *Amazon EC2 User Guide for Linux Instances*\. 

## Add lifecycle hooks<a name="adding-lifecycle-hooks"></a>

When you add a lifecycle hook to your Auto Scaling group, you can specify whether it should be run when instances launch or terminate in the Auto Scaling group\. 

**Topics**
+ [Add lifecycle hooks \(console\)](#adding-lifecycle-hooks-console)
+ [Add lifecycle hooks \(AWS CLI\)](#adding-lifecycle-hooks-aws-cli)

### Add lifecycle hooks \(console\)<a name="adding-lifecycle-hooks-console"></a>

Follow these steps to add a lifecycle hook to an existing Auto Scaling group\. You can specify whether the hook is used when the instances launch or terminate and how long to wait until the lifecycle hook is completed before abandoning or continuing\.

**To add a lifecycle hook**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Instance management** tab, in **Lifecycle hooks**, choose **Create lifecycle hook**\. \(Old console: The **Lifecycle Hooks** tab is where you can create a lifecycle hook\.\) 

1. To define a lifecycle hook, do the following:

   1. For **Lifecycle hook name**, specify a name for the lifecycle hook\.

   1. For **Lifecycle transition**, choose **Instance launch** or **Instance terminate**\. 

   1. Specify a timeout value for **Heartbeat timeout**, which allows you to control the amount of time for the instances to remain in a wait state\. The value must be from 30 to 7200 seconds\. During the timeout period, you can, for example, log on to a newly launched instance, and install applications or perform custom actions\. 

   1. For **Default result**, specify the action that the Auto Scaling group takes when the lifecycle hook timeout elapses or if an unexpected failure occurs\. You can choose to either ABANDON or CONTINUE\.

      If the instance is launching, CONTINUE indicates that your actions were successful, and that the Auto Scaling group can put the instance into service\. Otherwise, ABANDON indicates that your custom actions were unsuccessful, and that Auto Scaling can terminate the instance\. If the instance is terminating, both ABANDON and CONTINUE allow the instance to terminate\. However, ABANDON stops any remaining actions, such as other lifecycle hooks, and CONTINUE allows any other lifecycle hooks to complete\.

   1. \(Optional\) For **Notification metadata**, specify additional information that you want to include any time that Amazon EC2 Auto Scaling sends a message to the notification target\. 

1. Choose **Create**\.

### Add lifecycle hooks \(AWS CLI\)<a name="adding-lifecycle-hooks-aws-cli"></a>

Create and update lifecycle hooks using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command\.

To perform an action on scale out, use the following command\.

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-hook --auto-scaling-group-name my-asg \
  --lifecycle-transition autoscaling:EC2_INSTANCE_LAUNCHING
```

To perform an action on scale in, use the following command instead\.

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-hook --auto-scaling-group-name my-asg \
  --lifecycle-transition autoscaling:EC2_INSTANCE_TERMINATING
```

To receive notifications using Amazon SNS or Amazon SQS, you must specify a notification target and an IAM role\. For more information, see [Configuring notifications for Amazon EC2 Auto Scaling lifecycle hooks](configuring-lifecycle-hook-notifications.md)\. 

For example, add the following options to specify an SNS topic as the notification target\.

```
--notification-target-arn arn:aws:sns:region:123456789012:my-sns-topic --role-arn arn:aws:iam::123456789012:role/my-notification-role
```

The topic receives a test notification with the following key\-value pair\.

```
"Event": "autoscaling:TEST_NOTIFICATION"
```

## Complete a lifecycle hook custom action<a name="completing-lifecycle-hooks"></a>

When an Auto Scaling group responds to a scale\-out or scale\-in event, it puts the instance in a wait state and, depending on how the lifecycle hook is configured, sends a notification\.

**To complete a lifecycle hook custom action**

1. After receiving a notification, you can perform a custom action while the instance is in a wait state\.

1. If you need more time to complete the custom action, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) command to restart the timeout period and keep the instance in a wait state\. You can specify the lifecycle action token that you received with the notification, as shown in the following command\.

   ```
   aws autoscaling record-lifecycle-action-heartbeat --lifecycle-hook-name my-launch-hook \
     --auto-scaling-group-name my-asg --lifecycle-action-token bcd2f1b8-9a78-44d3-8a7a-4dd07d7cf635
   ```

   Alternatively, you can specify the ID of the instance you retrieved in the previous step, as shown in the following command\.

   ```
   aws autoscaling record-lifecycle-action-heartbeat --lifecycle-hook-name my-launch-hook \
     --auto-scaling-group-name my-asg --instance-id i-1a2b3c4d
   ```

1. If you finish the custom action before the timeout period ends, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command so that the Auto Scaling group can continue launching or terminating the instance\. You can specify the lifecycle action token, as shown in the following command\.

   ```
   aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE \
     --lifecycle-hook-name my-launch-hook --auto-scaling-group-name my-asg \
     --lifecycle-action-token bcd2f1b8-9a78-44d3-8a7a-4dd07d7cf635
   ```

   Alternatively, you can specify the ID of the instance, as shown in the following command\.

   ```
   aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE \
     --instance-id i-1a2b3c4d --lifecycle-hook-name my-launch-hook \
     --auto-scaling-group-name my-asg
   ```

## Test the notification<a name="testing-hook-notifications"></a>

To generate a notification for a launch event, update the Auto Scaling group by increasing the desired capacity of the Auto Scaling group by 1\. You receive a notification within a few minutes after instance launch\.

**To change the desired capacity \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Group details**, **Edit**\. \(Old console: On the **Details** tab, choose **Edit**\.\)

1. For **Desired capacity**, increase the current value by 1\. If this value exceeds **Maximum capacity**, you must also increase the value of **Maximum capacity** by 1\.

1. Choose **Update**\.

1. After a few minutes, you'll receive notification for the event\. If you do not need the additional instance that you launched for this test, you can decrease **Desired capacity** by 1\. After a few minutes, you'll receive notification for the event\.