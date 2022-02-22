# Adding lifecycle hooks<a name="adding-lifecycle-hooks"></a>

After your notification target is set up and ready to use, add the lifecycle hook so that the Event Notification can be used to perform your custom action when the corresponding lifecycle event occurs\. 

There are two types of lifecycle hooks that can be implemented: launch lifecycle hooks and termination lifecycle hooks\. Use a launch lifecycle hook to prepare instances for use or to delay instances from registering behind the load balancer before their configuration has been applied completely\. Use a termination lifecycle hook to prepare running instances to be shut down\. 

Pay attention to the following settings when creating your lifecycle hook: 
+ **Heartbeat timeout**: This setting specifies how much time must pass before the hook times out\. The range is from `30` to `7200` seconds\. The default value is one hour \(`3600` seconds\)\. During the timeout period, you can, for example, install applications or download logs or other data\.
+ **Default result**: This setting defines the action to take when the lifecycle hook timeout elapses or when an unexpected failure occurs\. You can choose to either *abandon* \(default\) or *continue*\.
  + If the instance is launching, continue indicates that your actions were successful, and that the Auto Scaling group can put the instance into service\. Otherwise, abandon indicates that your custom actions were unsuccessful, and that Amazon EC2 Auto Scaling can terminate the instance\. 
  + If the instance is terminating, both abandon and continue allow the instance to terminate\. However, abandon stops any remaining actions, such as other lifecycle hooks, and continue allows any other lifecycle hooks to complete\.

**Topics**
+ [Add lifecycle hooks \(console\)](#adding-lifecycle-hooks-console)
+ [Add lifecycle hooks \(AWS CLI\)](#adding-lifecycle-hooks-aws-cli)

## Add lifecycle hooks \(console\)<a name="adding-lifecycle-hooks-console"></a>

Follow these steps to add a lifecycle hook to an existing Auto Scaling group\. You can specify whether the hook is used when the instances launch or terminate, and how long to wait for the lifecycle hook to be completed before abandoning or continuing\.

**To add a lifecycle hook**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page\. 

1. On the **Instance management** tab, in **Lifecycle hooks**, choose **Create lifecycle hook**\.

1. To define a lifecycle hook, do the following:

   1. For **Lifecycle hook name**, specify a name for the lifecycle hook\.

   1. For **Lifecycle transition**, choose **Instance launch** or **Instance terminate**\. 

   1. Specify a timeout value for **Heartbeat timeout**, which allows you to control the amount of time for the instances to remain in a wait state\.

   1. For **Default result**, specify the action that the Auto Scaling group takes when the lifecycle hook timeout elapses or if an unexpected failure occurs\.

   1. \(Optional\) For **Notification metadata**, specify additional information that you want to include when Amazon EC2 Auto Scaling sends a message to the notification target\. 

1. Choose **Create**\.

## Add lifecycle hooks \(AWS CLI\)<a name="adding-lifecycle-hooks-aws-cli"></a>

Create and update lifecycle hooks using the [put\-lifecycle\-hook](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command\.

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

To receive notifications using Amazon SNS or Amazon SQS, you must specify a notification target and an IAM role\. For more information, see [Configuring a notification target for lifecycle notifications](configuring-lifecycle-hook-notifications.md)\. 

For example, add the following options to specify an SNS topic as the notification target\.

```
--notification-target-arn arn:aws:sns:region:123456789012:my-sns-topic --role-arn arn:aws:iam::123456789012:role/my-notification-role
```

The topic receives a test notification with the following key\-value pair\.

```
"Event": "autoscaling:TEST_NOTIFICATION"
```