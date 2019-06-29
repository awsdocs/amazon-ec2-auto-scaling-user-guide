# Replacing a Launch Configuration with a Launch Template<a name="replace-launch-config"></a>

When you edit an Auto Scaling group that has an existing launch configuration, you have the option of replacing the launch configuration with a launch template\. This lets you use launch templates with any Auto Scaling groups that you currently use\. In doing so, you can take advantage of the versioning and other features of launch templates\. 

After you replace the launch configuration for an Auto Scaling group, any new instances are launched using the new launch template, but existing instances are not affected\. 

**Prerequisites**  
Before you can replace a launch configuration in an Auto Scaling group, you must first create your launch template\. The easiest way to create a launch template is to copy it from the launch configuration\. For more information, see [Copying a Launch Configuration to a Launch Template](copy-launch-config.md)\.

**To replace the launch configuration for an Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Auto Scaling Groups**\.

1. Select the Auto Scaling group and choose **Details, Edit**\. 

1. For **Launch Instances Using**, choose the **Launch Template** option\.

1. For **Launch Template**, select your launch template\.

1. For **Launch Template Version**, select the launch template version, as needed\. After you create versions of a launch template, you can choose whether the Auto Scaling group uses the default or the latest version of the launch template when scaling out\.

1. When you have finished, choose **Save**\. 

**To replace a launch configuration using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html](https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)