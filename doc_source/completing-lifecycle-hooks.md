# Completing a lifecycle action<a name="completing-lifecycle-hooks"></a>

When an Auto Scaling group responds to a lifecycle event, it puts the instance in a wait state and sends an Event Notification\. You can perform a custom action while the instance is in a wait state\. 

## Completing a lifecycle action \(AWS CLI\)<a name="completing-lifecycle-hooks-aws-cli"></a>

The following procedure is for the command line interface and is not supported in the console\. Information that must be replaced, such as the instance ID or the name of an Auto Scaling group, are shown in italics\. 

**To complete a lifecycle action**

1. If you need more time to complete the custom action, use the [record\-lifecycle\-action\-heartbeat](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) command to restart the timeout period and keep the instance in a wait state\. You can specify the lifecycle action token that you received with the notification, as shown in the following command\.

   ```
   aws autoscaling record-lifecycle-action-heartbeat --lifecycle-hook-name my-launch-hook \
     --auto-scaling-group-name my-asg --lifecycle-action-token bcd2f1b8-9a78-44d3-8a7a-4dd07d7cf635
   ```

   Alternatively, you can specify the ID of the instance that you retrieved in the previous step, as shown in the following command\.

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