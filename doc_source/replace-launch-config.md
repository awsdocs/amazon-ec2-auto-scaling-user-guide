# Replacing a launch configuration with a launch template<a name="replace-launch-config"></a>

When you edit an Auto Scaling group that has an existing launch configuration, you have the option of replacing the launch configuration with a launch template\. This lets you use launch templates with any Auto Scaling groups that you currently use\. In doing so, you can take advantage of the versioning and other features of launch templates\. 

After you replace the launch configuration for an Auto Scaling group, any new instances are launched using the new launch template, but existing instances are not affected\. To update the existing instances, terminate them so that they are replaced by your Auto Scaling group, or allow automatic scaling to gradually replace older instances with newer instances based on your [termination policies](as-instance-termination.md)\. 

**Note**  
With the maximum instance lifetime and instance refresh features, you can also replace all instances in the Auto Scaling group to launch new instances that use the launch template\. For more information, see [Replacing Auto Scaling instances based on maximum instance lifetime](asg-max-instance-lifetime.md) and [Replacing Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

**Prerequisites**  
Before you can replace a launch configuration in an Auto Scaling group, you must first create your launch template\. The easiest way to create a launch template is to copy it from the launch configuration\. For more information, see [Copying a launch configuration to a launch template](copy-launch-config.md)\.

When you replace a launch configuration with a launch template, your `ec2:RunInstances` permissions are checked\. If you are attempting to use a launch template and you do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. For information about the required IAM permissions, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

**To replace the launch configuration for an Auto Scaling group \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\. 

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Launch configuration**, **Edit**\.

1. Choose **Switch to launch template**\.

1. For **Launch template**, select your launch template\.

1. For **Version**, select the launch template version, as needed\. After you create versions of a launch template, you can choose whether the Auto Scaling group uses the default or the latest version of the launch template when scaling out\.

1. When you have finished, choose **Update**\. 

**To replace the launch configuration for an Auto Scaling group \(old console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\. 

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. Choose **Details, Edit**\. 

1. For **Launch Instances Using**, choose the **Launch Template** option\.

1. For **Launch Template**, select your launch template\.

1. For **Launch Template Version**, select the launch template version, as needed\. After you create versions of a launch template, you can choose whether the Auto Scaling group uses the default or the latest version of the launch template when scaling out\.

1. When you have finished, choose **Save**\. 

**To replace a launch configuration using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html](https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)