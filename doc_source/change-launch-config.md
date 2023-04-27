# Change the launch configuration for an Auto Scaling group<a name="change-launch-config"></a>

**Important**  
We provide information about launch configurations for customers who have not yet migrated from launch configurations to launch templates\. For information about migrating existing Auto Scaling groups to launch templates, see [Migrate to launch templates](launch-templates.md#migrate-to-launch-templates) and [Amazon EC2 Auto Scaling will no longer add support for new EC2 features to Launch Configurations](http://aws.amazon.com/blogs/compute/amazon-ec2-auto-scaling-will-no-longer-add-support-for-new-ec2-features-to-launch-configurations/) on the AWS Compute Blog\.

An Auto Scaling group is associated with one launch configuration at a time, and you can't modify a launch configuration after you've created it\. To change the launch configuration for an Auto Scaling group, use an existing launch configuration as the basis for a new launch configuration\. Then, update the Auto Scaling group to use the new launch configuration\.

After you change the launch configuration for an Auto Scaling group, any new instances are launched using the new configuration options, but existing instances are not affected\. To update the existing instances, terminate them so that they are replaced by your Auto Scaling group, or allow auto scaling to gradually replace older instances with newer instances based on your [termination policies](as-instance-termination.md)\. 

**Note**  
You can also replace all instances in the Auto Scaling group to launch new instances that use the new launch configuration\. For more information, see [Replace Auto Scaling instances](ec2-auto-scaling-group-replacing-instances.md)\.

**To change the launch configuration for an Auto Scaling group \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Launch Configurations**\.

1. Select the launch configuration and choose **Actions**, **Copy launch configuration**\. This sets up a new launch configuration with the same options as the original, but with "Copy" added to the name\.

1. On the **Copy Launch Configuration** page, edit the configuration options as needed and choose **Create launch configuration**\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Launch configuration**, **Edit**\.

1. For **Launch configuration**, select the new launch configuration\.

1. When you have finished, choose **Update**\. 

**To change the launch configuration for an Auto Scaling group \(AWS CLI\)**

1. Describe the current launch configuration using the [describe\-launch\-configurations](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command\.

1. Create a new launch configuration using the [create\-launch\-configuration](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command\.

1. Update the launch configuration for the Auto Scaling group using the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command with the `--launch-configuration-names` parameter\.

**To change the launch configuration for an Auto Scaling group \(Tools for Windows PowerShell\)**

1. Describe the current launch configuration using the [Get\-ASLaunchConfiguration](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-ASLaunchConfiguration.html) command\.

1. Create a new launch configuration using the [New\-ASLaunchConfiguration](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASLaunchConfiguration.html) command\.

1. Update the launch configuration for the Auto Scaling group using the [Update\-ASAutoScalingGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html) command with the `-LaunchConfigurationName` parameter\.