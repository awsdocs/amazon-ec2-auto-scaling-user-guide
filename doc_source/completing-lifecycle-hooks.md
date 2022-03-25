# Completing a lifecycle action<a name="completing-lifecycle-hooks"></a>

When an Auto Scaling group responds to a lifecycle event, it puts the instance in a wait state and sends an event notification\. You can perform a custom action while the instance is in a wait state\.

**Topics**
+ [Completing a lifecycle action \(manual\)](#completing-lifecycle-hooks-aws-cli)
+ [Completing a lifecycle action \(automatic\)](#completing-lifecycle-hooks-automatic)

## Completing a lifecycle action \(manual\)<a name="completing-lifecycle-hooks-aws-cli"></a>

The following procedure is for the command line interface and is not supported in the console\. Information that must be replaced, such as the instance ID or the name of an Auto Scaling group, are shown in italics\. 

**To complete a lifecycle action \(AWS CLI\)**

1. If you need more time to complete the custom action, use the [record\-lifecycle\-action\-heartbeat](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) command to restart the timeout period and keep the instance in a wait state\. For example, if the timeout period is one hour, and you call this command after 30 minutes, the instance remains in a wait state for an additional hour, or a total of 90 minutes\. 

   You can specify the lifecycle action token that you received with the [notification](prepare-for-lifecycle-notifications.md#notification-message-example), as shown in the following command\.

   ```
   aws autoscaling record-lifecycle-action-heartbeat --lifecycle-hook-name my-launch-hook \
     --auto-scaling-group-name my-asg --lifecycle-action-token bcd2f1b8-9a78-44d3-8a7a-4dd07d7cf635
   ```

   Alternatively, you can specify the ID of the instance that you received with the [notification](prepare-for-lifecycle-notifications.md#notification-message-example), as shown in the following command\.

   ```
   aws autoscaling record-lifecycle-action-heartbeat --lifecycle-hook-name my-launch-hook \
     --auto-scaling-group-name my-asg --instance-id i-1a2b3c4d
   ```

1. If you finish the custom action before the timeout period ends, use the [complete\-lifecycle\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command so that the Auto Scaling group can continue launching or terminating the instance\. You can specify the lifecycle action token, as shown in the following command\.

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

## Completing a lifecycle action \(automatic\)<a name="completing-lifecycle-hooks-automatic"></a>

If you have a user data script that configures your instances after they launch, you do not need to manually complete lifecycle actions\. You can add the [complete\-lifecycle\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command to the script\. The script can retrieve the instance ID from the instance metadata and signal Amazon EC2 Auto Scaling when the bootstrap scripts have completed successfully\. 

If you are not doing so already, update your script to retrieve the instance ID of the instance from the instance metadata\. For more information, see [Retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) in the *Amazon EC2 User Guide for Linux Instances*\.

If you use Lambda, you can also set up a callback in your function's code to let the lifecycle of the instance proceed if the custom action is successful\. For more information, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\.