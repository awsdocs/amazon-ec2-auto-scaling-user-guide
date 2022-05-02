# Add checkpoints to an instance refresh<a name="asg-adding-checkpoints-instance-refresh"></a>

When using an instance refresh, you have the option to replace instances in phases, so that you can perform verifications on your instances as you go\. To do a phased replacement, you add checkpoints, which are points in time where the instance refresh pauses\. Using checkpoints gives you greater control over how you choose to update your Auto Scaling group\. It helps you to confirm that your application will function in a reliable, predictable manner\.

Amazon EC2 Auto Scaling emits events for each checkpoint\. If you add an EventBridge rule to send the events to a target such as Amazon SNS, you can be notified when you can run the required verifications\. For more information, see [Create EventBridge rules for instance refresh events](monitor-events-eventbridge-sns.md)\.

**Topics**
+ [Considerations](#instance-refresh-checkpoints-considerations)
+ [Enable checkpoints \(console\)](#enable-checkpoints-console)
+ [Enable checkpoints \(AWS CLI\)](#enable-checkpoints-cli)

## Considerations<a name="instance-refresh-checkpoints-considerations"></a>

Keep the following considerations in mind when using checkpoints:
+ A checkpoint is reached when the number of instances replaced reaches the percentage threshold that is defined for the checkpoint\. The percentage of instances replaced can be equal to or higher than the percentage threshold, but not lower\. 
+ After a checkpoint is reached, the overall percentage complete doesn't display the latest status until after the instances finish warming up\. 

  For example, assume that your Auto Scaling group has 10 instances\. Your checkpoint percentages are `[20,50]` with a checkpoint delay of 15 minutes and a minimum healthy percentage of 80 percent\. Your group makes the following replacements:
  + 0:00: Two old instances are replaced with new ones\. 
  + 0:10: Two new instances finish warming up\. 
  + 0:25: Two old instances are replaced with new ones\. \(To maintain the minimum healthy percentage, only two instances are replaced\.\)
  + 0:35: Two new instances finish warming up\. 
  + 0:35: One old instance is replaced with a new one\.
  + 0:45: One new instance finishes warming up\.

  At 0:35, the operation stops launching new instances\. The percentage complete doesn't accurately reflect the number of completed replacements yet \(50 percent\), because the new instance isn't done warming up\. After the new instance completes its warm\-up period at 0:45, the percentage complete shows 50 percent\.
+ Because checkpoints are based on percentages, the number of instances to replace changes with the size of the group\. When a scale\-out activity occurs and the size of the group increases, an in\-progress operation could reach a checkpoint again\. If that happens, Amazon EC2 Auto Scaling sends another notification and repeats the wait time between checkpoints before continuing\.
+ It's possible to skip a checkpoint under certain circumstances\. For example, suppose that your Auto Scaling group has two instances and your checkpoint percentages are `[10,40,100]`\. After the first instance is replaced, Amazon EC2 Auto Scaling calculates that 50 percent of the group was replaced\. Because 50 percent is higher than the first two checkpoints, it skips the first checkpoint \(`10`\) and sends a notification for the second checkpoint \(`40`\)\.
+ Canceling the operation stops any further replacements from being made\. If you cancel the operation or it fails before reaching the last checkpoint, any instances that were already replaced are not rolled back to their previous configuration\.
+ For a partial refresh, when you rerun the operation, Amazon EC2 Auto Scaling doesn't restart from the point of the last checkpoint, nor does it stop when only the old instances are replaced\. However, it targets old instances for replacement first, before targeting new instances\. 

## Enable checkpoints \(console\)<a name="enable-checkpoints-console"></a>

You can enable checkpoints before starting an instance refresh to replace instances using an incremental or phased approach\. This provides additional time for verification\.

**To start an instance refresh that uses checkpoints**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. On the **Instance refresh** tab, in **Active instance refresh**, choose **Start instance refresh**\.

1. On the **Start instance refresh** page, enter the values for **Minimum healthy percentage** and **Instance warmup**\. 

1. Select the **Enable checkpoints** check box\.

   This displays a box where you can define the percentage threshold for the first checkpoint\. 

1. For **Proceed until \_\_\_\_ % of the group is refreshed**, enter a number \(1–100\)\. This sets the percentage for the first checkpoint\. 

1. To add another checkpoint, choose **Add checkpoint** and then define the percentage for the next checkpoint\.

1. To specify how long Amazon EC2 Auto Scaling waits after a checkpoint is reached, update the fields in **Wait for `1` `hour` between checkpoints**\. The time unit can be hours, minutes, or seconds\.

1. If you are finished with your instance refresh selections, choose **Start**\. 

## Enable checkpoints \(AWS CLI\)<a name="enable-checkpoints-cli"></a>

To start an instance refresh with checkpoints enabled using the AWS CLI, you need a configuration file that defines the following parameters:
+ `CheckpointPercentages`: Specifies threshold values for the percentage of instances to be replaced\. These threshold values provide the checkpoints\. When the percentage of instances that are replaced and warmed up reaches one of the specified thresholds, the operation waits for a specified period of time\. You specify the number of seconds to wait in `CheckpointDelay`\. When the specified period of time has passed, the instance refresh continues until it reaches the next checkpoint \(if applicable\)\.
+ `CheckpointDelay`: Specifies the amount of time, in seconds, to wait after a checkpoint is reached before continuing\. Choose a time period that provides enough time to perform your verifications\.

The last value shown in the `CheckpointPercentages` array describes the percentage of the Auto Scaling group that needs to be successfully replaced\. The operation transitions to `Successful` after this percentage is successfully replaced and each instance is warmed up and ready to serve traffic again\. 

**To create multiple checkpoints**  
To create multiple checkpoints, use the following example [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command\. This example configures an instance refresh that initially refreshes one percent of the Auto Scaling group\. After waiting 10 minutes, it then refreshes the next 19 percent and waits another 10 minutes\. Finally, it refreshes the rest of the group before concluding the operation\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`:

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 60,
      "MinHealthyPercentage": 80,
      "CheckpointPercentages": [1,20,100],
      "CheckpointDelay": 600
    }
}
```

**To create a single checkpoint**  
To create a single checkpoint, use the following example [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command\. This example configures an instance refresh that initially refreshes 20 percent of the Auto Scaling group\. After waiting 10 minutes, it then refreshes the rest of the group before concluding the operation\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`:

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 60,
      "MinHealthyPercentage": 80,
      "CheckpointPercentages": [20,100],
      "CheckpointDelay": 600
    }
}
```

**To partially refresh the Auto Scaling group**  
To replace only a portion of your Auto Scaling group and then stop completely, use the following example [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command\. This example configures an instance refresh that initially refreshes one percent of the Auto Scaling group\. After waiting 10 minutes, it then refreshes the next 19 percent before concluding the operation\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`:

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 60,
      "MinHealthyPercentage": 80,
      "CheckpointPercentages": [1,20],
      "CheckpointDelay": 600
    }
}
```