# Undo changes with a rollback<a name="instance-refresh-rollback"></a>

You can roll back an instance refresh that is still in progress\. You can't roll it back after it's finished\. You can, however, update your Auto Scaling group again by starting a new instance refresh\.

When rolling back, Amazon EC2 Auto Scaling replaces the instances that have been deployed so far\. The new instances match the configuration that you last saved on the Auto Scaling group before starting the instance refresh\.

Amazon EC2 Auto Scaling provides the following ways to roll back:
+ Manual rollback: You start a rollback manually to reverse what was deployed up to the rollback point\.
+ Auto rollback: Amazon EC2 Auto Scaling automatically reverses what was deployed if the instance refresh fails for some reason\.

**Topics**
+ [Prerequisites](#instance-refresh-rollback-prerequisites)
+ [Manually start a rollback \(console\)](#instance-refresh-manual-rollback-console)
+ [Manually start a rollback \(AWS CLI\)](#instance-refresh-manual-rollback-cli)
+ [Start an instance refresh with auto rollback](#instance-refresh-using-auto-rollback)

## Prerequisites<a name="instance-refresh-rollback-prerequisites"></a>

To use the rollback features, you must do the following:
+ Specify a desired configuration\.
+ Make sure that the configuration that you last saved on the Auto Scaling group is in a stable state\. If it is not in a stable state, the rollback workflow will still occur, but it will eventually fail\. Until you resolve the issue, the Auto Scaling group might be in a failed state where it can no longer launch instances successfully\. This might impact the availability of the service or application\.
+ Make sure that it's possible to roll back to your Auto Scaling group's current launch template\. If the Auto Scaling group has an incompatible launch template or launch template version, you receive an error when you start a manual rollback or an instance refresh with auto rollback enabled\. Incompatible launch templates use an AMI alias from the AWS Systems Manager Parameter Store\. Incompatible launch template versions include `$Latest` or `$Default`\.

## Manually start a rollback \(console\)<a name="instance-refresh-manual-rollback-console"></a>

**To manually start a rollback of an instance refresh**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to the Auto Scaling group\.

1. On the **Instance refresh** tab, in **Active instance refresh**, choose **Actions**, **Start rollback**\.

1. When prompted for confirmation, choose **Confirm**\.

## Manually start a rollback \(AWS CLI\)<a name="instance-refresh-manual-rollback-cli"></a>

**To manually start a rollback of an instance refresh**  
Use the [rollback\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/rollback-instance-refresh.html) command from the AWS CLI and provide the Auto Scaling group name\. 

```
aws autoscaling rollback-instance-refresh --auto-scaling-group-name my-asg
```

Example output:

```
{
    "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
}
```

**Tip**  
If this command throws an error, make sure that you have updated the AWS CLI locally to the latest version\.

## Start an instance refresh with auto rollback<a name="instance-refresh-using-auto-rollback"></a>

By default, if the instance refresh fails, it doesn't roll back the changes that it applied to new instances\. You can change the behavior so that a rollback occurs when the instance refresh fails\.

This section includes AWS CLI instructions for starting an instance refresh with auto rollback\. For instructions on using the console, see [Start an instance refresh \(console\)](start-instance-refresh.md#start-instance-refresh-console)\.

**To start an instance refresh with auto rollback \(AWS CLI\)**  
Use the [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command with the auto rollback option in the preferences set to `true`\. Also, provide the Auto Scaling group name and desired configuration\. 

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`\.

```
{
    "AutoScalingGroupName": "my-asg",
    "DesiredConfiguration": {
      "LaunchTemplate": {
          "LaunchTemplateId": "lt-068f72b729example",
          "Version": "$Default"
       }
    },
    "Preferences": {
      "AutoRollback": true
    }
}
```

If successful, the command returns output similar to the following\.

```
{
  "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
}
```

**Tip**  
If this command throws an error, make sure that you have updated the AWS CLI locally to the latest version\.