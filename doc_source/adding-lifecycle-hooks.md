# Adding lifecycle hooks<a name="adding-lifecycle-hooks"></a>

To put your Auto Scaling instances into a wait state and perform custom actions on them, you can add lifecycle hooks to your Auto Scaling group\. Custom actions are performed as the instances launch or before they terminate\. Instances remain in a wait state until you either complete the lifecycle action, or the timeout period ends\.

After you create an Auto Scaling group from the AWS Management Console, you can add one or more lifecycle hooks to it, up to a total of 50 lifecycle hooks\. You can also use the AWS CLI, AWS CloudFormation, or an SDK to add lifecycle hooks to an Auto Scaling group as you are creating it\.

By default, when you add a lifecycle hook in the console, Amazon EC2 Auto Scaling sends lifecycle event notifications to Amazon EventBridge\. Using EventBridge or a user data script is a recommended best practice\. To create a lifecycle hook that sends notifications directly to Amazon SNS or Amazon SQS, you can use the [put\-lifecycle\-hook](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command, as shown in the examples in this topic\.

**Topics**
+ [Add lifecycle hooks \(console\)](#adding-lifecycle-hooks-console)
+ [Add lifecycle hooks \(AWS CLI\)](#adding-lifecycle-hooks-aws-cli)

## Add lifecycle hooks \(console\)<a name="adding-lifecycle-hooks-console"></a>

Follow these steps to add a lifecycle hook to your Auto Scaling group\. To create lifecycle hooks for scaling out \(instances launching\) and scaling in \(instances terminating\), you must create two separate hooks\. 

Before you begin, confirm that you have set up a custom action, as described in [Prepare to add a lifecycle hook to your Auto Scaling group](prepare-for-lifecycle-notifications.md)\.

**To add a lifecycle hook**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page\. 

1. On the **Instance management** tab, in **Lifecycle hooks**, choose **Create lifecycle hook**\.

1. To define a lifecycle hook, do the following:

   1. For **Lifecycle hook name**, specify a name for the lifecycle hook\.

   1. For **Lifecycle transition**, choose **Instance launch** or **Instance terminate**\. 

   1. For **Heartbeat timeout**, specify the amount of time for instances to remain in a wait state when scaling out or scaling in before the hook times out\. The range is from `30` to `7200` seconds\. 
**Note**  
By default, the timeout is `3600` seconds \(one hour\)\. Setting a long timeout period provides more time for the custom action to complete\. If you finish before the timeout period ends, you can send the [complete\-lifecycle\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command to allow the instance to proceed to the next state\. 

   1. For **Default result**, specify the action to take when the lifecycle hook timeout elapses or when an unexpected failure occurs\. You can choose to either *abandon* \(default\) or *continue*\.
      + If the instance is launching, continue indicates that your actions were successful, and that Amazon EC2 Auto Scaling can put the instance into service\. Otherwise, abandon indicates that your custom actions were unsuccessful, and that Amazon EC2 Auto Scaling can terminate the instance\. 
      + If the instance is terminating, both abandon and continue allow the instance to terminate\. However, abandon stops any remaining actions, such as other lifecycle hooks, and continue allows any other lifecycle hooks to complete\.

   1. \(Optional\) For **Notification metadata**, specify additional information that you want to include when Amazon EC2 Auto Scaling sends a message to the notification target\. 

1. Choose **Create**\.

## Add lifecycle hooks \(AWS CLI\)<a name="adding-lifecycle-hooks-aws-cli"></a>

Create and update lifecycle hooks using the [put\-lifecycle\-hook](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command\.

To perform an action on scale out, use the following command\.

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-launch-hook  \
  --auto-scaling-group-name my-asg \
  --lifecycle-transition autoscaling:EC2_INSTANCE_LAUNCHING
```

To perform an action on scale in, use the following command instead\.

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-termination-hook  \
  --auto-scaling-group-name my-asg \
  --lifecycle-transition autoscaling:EC2_INSTANCE_TERMINATING
```

To receive notifications using Amazon SNS or Amazon SQS, add the `--notification-target-arn` and `--role-arn` options\.

The following example creates a lifecycle hook that specifies an SNS topic named `my-sns-topic` as the notification target\.

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-termination-hook  \
  --auto-scaling-group-name my-asg \
  --lifecycle-transition autoscaling:EC2_INSTANCE_TERMINATING \
  --notification-target-arn arn:aws:sns:region:123456789012:my-sns-topic \
  --role-arn arn:aws:iam::123456789012:role/my-notification-role
```

The topic receives a test notification with the following key\-value pair\.

```
"Event": "autoscaling:TEST_NOTIFICATION"
```

By default, the [put\-lifecycle\-hook](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command creates a lifecycle hook with a heartbeat timeout of `3600` seconds \(one hour\)\. 

To change the heartbeat timeout for an existing lifecycle hook, add the `--heartbeat-timeout` option, as shown in the following example\.

```
aws autoscaling put-lifecycle-hook --lifecycle-hook-name my-termination-hook \
  --auto-scaling-group-name my-asg --heartbeat-timeout 120
```

If an instance is already in a wait state, you can prevent the lifecycle hook from timing out by recording a heartbeat, using the [record\-lifecycle\-action\-heartbeat](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) CLI command\. This extends the timeout period by the timeout value specified when you created the lifecycle hook\. If you finish before the timeout period ends, you can send the [complete\-lifecycle\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) CLI command to allow the instance to proceed to the next state\. For more information and examples, see [Completing a lifecycle action](completing-lifecycle-hooks.md)\.