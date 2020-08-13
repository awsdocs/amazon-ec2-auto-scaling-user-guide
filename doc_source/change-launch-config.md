# Changing the launch configuration for an Auto Scaling group<a name="change-launch-config"></a>

An Auto Scaling group is associated with one launch configuration at a time, and you can't modify a launch configuration after you've created it\. To change the launch configuration for an Auto Scaling group, use an existing launch configuration as the basis for a new launch configuration\. Then, update the Auto Scaling group to use the new launch configuration\.

After you change the launch configuration for an Auto Scaling group, any new instances are launched using the new configuration options, but existing instances are not affected\. To update the existing instances, terminate them so that they are replaced by your Auto Scaling group, or allow automatic scaling to gradually replace older instances with newer instances based on your [termination policies](as-instance-termination.md)\. 

**Note**  
With the maximum instance lifetime and instance refresh features, you can also replace all instances in the Auto Scaling group to launch new instances that use the new launch configuration\. For more information, see [Replacing Auto Scaling instances based on maximum instance lifetime](asg-max-instance-lifetime.md) and [Replacing Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

**To change the launch configuration for an Auto Scaling group \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\.

1. Select the launch configuration and choose **Actions**, **Copy launch configuration**\. This sets up a new launch configuration with the same options as the original, but with "Copy" added to the name\.

1. On the **Copy Launch Configuration** page, edit the configuration options as needed and choose **Create launch configuration**\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Launch configuration**, **Edit**\.

1. For **Launch configuration**, select the new launch configuration\.

1. When you have finished, choose **Update**\. 

**To change the launch configuration for an Auto Scaling group \(old console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\.

1. Select the launch configuration and choose **Actions**, **Copy launch configuration**\. This sets up a new launch configuration with the same options as the original, but with "Copy" added to the name\.

1. On the **Copy Launch Configuration** page, edit the configuration options as needed and choose **Create launch configuration**\.

1. On the confirmation page, choose **View your Auto Scaling groups**\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. Choose **Details**, **Edit**\. 

1. Select the new launch configuration from **Launch Configuration** and choose **Save**\.

**To change the launch configuration for an Auto Scaling group \(AWS CLI\)**

1. Describe the current launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command\.

1. Create a new launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command\.

1. Update the launch configuration for the Auto Scaling group using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command with the `--launch-configuration-names` parameter\.

**To change the launch configuration for an Auto Scaling group \(Tools for Windows PowerShell\)**

1. Describe the current launch configuration using the [https://docs.aws.amazon.com/powershell/latest/reference/items/Get-ASLaunchConfiguration.html](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-ASLaunchConfiguration.html) command\.

1. Create a new launch configuration using the [https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASLaunchConfiguration.html](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASLaunchConfiguration.html) command\.

1. Update the launch configuration for the Auto Scaling group using the [https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html](https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html) command with the `-LaunchConfigurationName` parameter\.