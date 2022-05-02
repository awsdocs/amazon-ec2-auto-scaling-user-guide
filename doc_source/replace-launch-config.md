# Replace a launch configuration with a launch template<a name="replace-launch-config"></a>

When you edit an Auto Scaling group that has an existing launch configuration, you have the option of replacing the launch configuration with a launch template\. This lets you use launch templates with any Auto Scaling groups that you currently use\. In doing so, you can take advantage of versioning and other features of launch templates\. 

After you replace the launch configuration for an Auto Scaling group, any new instances are launched using the new launch template\. Existing instances are not affected\. To update the existing instances, terminate them so that they are replaced by your Auto Scaling group, or allow automatic scaling to gradually replace earlier instances with newer instances based on your [termination policies](as-instance-termination.md)\. 

**Note**  
With the instance refresh feature, you can replace the instances in the Auto Scaling group to launch new instances that use the launch template immediately\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

**Prerequisites**  
Before you can replace a launch configuration in an Auto Scaling group, you must first create your launch template\. A basic way to create a launch template is to copy it from the launch configuration\. For more information, see [Copy launch configurations to launch templates](copy-launch-config.md)\.

If you switch your Auto Scaling group from using a launch configuration, be sure that your permissions are up to date\. In order to use a launch template, you need specific [permissions](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html#launch-templates-permissions)\.

**To replace the launch configuration for an Auto Scaling group \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\. 

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Launch configuration**, **Edit**\.

1. Choose **Switch to launch template**\.

1. For **Launch template**, select your launch template\.

1. For **Version**, select the launch template version, as needed\. After you create versions of a launch template, you can choose whether the Auto Scaling group uses the default or the latest version of the launch template when scaling out\.

1. When you have finished, choose **Update**\. 

**To replace a launch configuration using the command line**

You can use one of the following commands:
+ [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) \(AWS CLI\)
+ [Update\-ASAutoScalingGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/Update-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)

For examples of using a CLI command to update an Auto Scaling group to use a launch template, see [Update an Auto Scaling group to use a launch template](examples-launch-templates-aws-cli.md#update-asg-launch-template-cli)\.