# Replace a launch configuration with a launch template<a name="replace-launch-config"></a>

**Note**  
To migrate all your existing Auto Scaling groups to launch templates, see [Migrate to launch templates](launch-templates.md#migrate-to-launch-templates)\. This section also shows you how to identify whether you have Auto Scaling groups that are still using launch configurations using the AWS CLI\.

When an Auto Scaling group is edited, you can specify a launch template for that Auto Scaling group to replace a launch configuration\.

After you replace the launch configuration, any new instances are launched using the new launch template\. Existing instances are not affected\. To update the existing instances, you can allow automatic scaling to gradually replace existing instances with new instances based on the group's [termination policies](as-instance-termination.md), or you can terminate them\. Amazon EC2 Auto Scaling immediately starts launching new instances to replace the instances that you terminated\. Alternatively, you can start an instance refresh to replace the instances\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

**Prerequisites**

Create a launch template that includes the parameters required to launch an EC2 instance, such as the Amazon Machine Image \(AMI\) and security groups\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\. You can also copy an existing launch configuration\. For more information, see [Copy launch configurations to launch templates](copy-launch-config.md)\.

Verify that you have the permissions required to use a launch template\. For more information, see [Permissions](launch-templates.md#launch-templates-permissions)\.

**To replace the launch configuration for an Auto Scaling group \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

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