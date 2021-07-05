# Adding checkpoints to an instance refresh<a name="asg-adding-checkpoints-instance-refresh"></a>

When using an instance refresh, you have the option to replace the entire Auto Scaling group in one continuous operation\. However, you might prefer to replace the group in phases, so that you can perform verifications on your instances as you go\. To do a phased replacement, you add checkpoints, which are points in time where the instance refresh pauses\. Using checkpoints gives you greater control over how you choose to update your Auto Scaling group, and it helps you to ensure that your application will function in a reliable, predictable manner\.

To enable checkpoints for an instance refresh, add the following refresh preferences to the configuration file that defines the instance refresh parameters when you use the AWS CLI:
+ `CheckpointPercentages`: Specifies threshold values for the percentage of instances to be replaced\. These threshold values provide the checkpoints\. When the percentage of instances that are replaced and warmed up reaches one of the specified thresholds, the operation waits for a specified period of time\. You specify the number of seconds to wait in `CheckpointDelay`\. When the specified period of time has passed, the instance refresh continues until it reaches the next checkpoint \(if applicable\)\.
+ `CheckpointDelay`: The amount of time, in seconds, to wait after a checkpoint is reached before continuing\. Choose a time period that allows you enough time to perform your validations\.

The percentage of the Auto Scaling group that needs to be successfully replaced is indicated by the last value shown in the `CheckpointPercentages` array\. The operation doesn't transition to `Successful` until this percentage of the group is replaced successfully, and each instance is warmed up and ready to start serving traffic again\. 

Amazon EC2 Auto Scaling emits events for each checkpoint\. If you add an EventBridge rule to send the events to a target such as Amazon SNS, you can be notified when you can run the required validations\. For more information, see [Creating EventBridge rules for instance refresh events](cloud-watch-events.md#monitor-events-eventbridge-sns)\.

## Setting instance refresh checkpoint preferences \(console\)<a name="setting-checkpoint-preferences-console"></a>

You can configure checkpoints in the preferences for an instance refresh\.

**To start an instance refresh that uses checkpoints**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page\. 

1. On the **Instance refresh** tab, in **Instance refreshes**, choose **Start instance refresh**\.

1. In the **Start instance refresh** dialog box, specify the applicable values for **Minimum healthy percentage** and **Instance warmup**\. 

1. Select the **Enable instance refresh with checkpoints** check box\.

   This displays a box where you can define the percentage threshold for the first checkpoint\. 

1. To set a percentage for the first checkpoint, enter a number from 1 to 100 in the number field in **Proceed until \_\_\_\_ % of the group is refreshed**\. 

1. To add another checkpoint, choose **Add checkpoint** and then define the percentage for the next checkpoint\.

1. To specify how long Amazon EC2 Auto Scaling waits after a checkpoint is reached, update the fields in **Wait for `1` `hour` between checkpoints**\. The time unit can be hours, minutes, or seconds\.

1. Choose **Start**\. 

## Setting instance refresh checkpoint preferences \(AWS CLI\)<a name="setting-checkpoint-preferences-cli"></a>

**To create multiple checkpoints**  
To create multiple checkpoints, use the following example [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command\. This example configures an instance refresh that refreshes 1 percent of the Auto Scaling group initially, waits 10 minutes, then refreshes the next 19 percent, waits 10 minutes, and then refreshes the rest of the group before succeeding the operation\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`\.

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 400,
      "MinHealthyPercentage": 80,
      "CheckpointPercentages": [1,20,100],
      "CheckpointDelay": 600
    }
}
```

**To create a single checkpoint**  
To create a single checkpoint, use the following example [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command\. This example configures an instance refresh that refreshes 20 percent of the Auto Scaling group initially, waits 10 minutes, and then refreshes the rest of the group before concluding the operation\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`\.

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 400,
      "MinHealthyPercentage": 80,
      "CheckpointPercentages": [20,100],
      "CheckpointDelay": 600
    }
}
```

**To only partially refresh the Auto Scaling group**  
To replace a portion of your Auto Scaling group and then stop completely, use the following example [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command\. This example configures an instance refresh that refreshes 1 percent of the Auto Scaling group initially, waits 10 minutes, and then refreshes the next 19 percent before concluding the operation\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`\.

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 400,
      "MinHealthyPercentage": 80,
      "CheckpointPercentages": [1,20],
      "CheckpointDelay": 600
    }
}
```

## Key points about checkpoints<a name="instance-refresh-checkpoints-considerations"></a>

The following are key points to note about using checkpoints:
+ A checkpoint is reached when the number of instances replaced reaches the percentage threshold that is defined for the checkpoint\. The percentage of instances replaced could be at or above, but not below, the percentage threshold\. 
+ After a checkpoint is reached, the overall percentage complete does not immediately display the latest status until the instances finish warming up\. 

  For example, assume your Auto Scaling group has 10 instances and makes the following replacements\. Your checkpoint percentages are `[20,50]` with a checkpoint delay of 15 minutes and a minimum healthy percentage of 80%\.
  + 0:00: 2 old instances are replaced\. 
  + 0:10: 2 new instances finish warming up\. 
  + 0:25: 2 old instances are replaced\. Only two instances are replaced to maintain the minimum healthy percentage\.
  + 0:35: 2 new instances finish warming up\. 
  + 0:35: 1 old instance is replaced\.
  + 0:45: 1 new instance finishes warming up\.

  At 0:35, the operation will stop launching new instances even though the percentage complete won't yet accurately reflect the number of replacements complete at 50% until 10 minutes later when the new instance completes its warm\-up period\.
+ Because checkpoints are based on percentages, the number of instances to replace to reach a checkpoint changes with the size of the group\. This means that when a scale\-out activity occurs and the size of the group increases, an in\-progress operation could reach a checkpoint again\. If that happens, Amazon EC2 Auto Scaling sends another notification and repeats the wait time between checkpoints before continuing\.
+ It's possible to skip a checkpoint under certain circumstances\. For example, suppose your Auto Scaling group has 2 instances and your checkpoint percentages are `[10,40,100]`\. After the first instance is replaced, Amazon EC2 Auto Scaling calculates that 50% of the group has been replaced\. Because 50% is higher than the first two checkpoints, it skips the first checkpoint \(`10`\) and sends a notification for the second checkpoint \(`40`\)\.
+ Cancelling the operation stops any further replacements from being made\. If you cancel the operation or it fails before reaching the last checkpoint, any instances that were already replaced are not rolled back to their previous configuration\. 
+ In the case of a partial refresh, when you rerun the operation, Amazon EC2 Auto Scaling doesn't restart from the point of the last checkpoint and stop when only the old instances are replaced\. However, it will target old instances for replacement first before targeting new instances\. 