# Copying a Launch Configuration to a Launch Template<a name="copy-launch-config"></a>

Use the following procedure to copy the options from an existing launch configuration to create a new launch template\.

You can create launch templates from existing launch configurations to make it easy for you to update your Auto Scaling groups to use launch templates\. Like launch configurations, launch templates can contain all or some of the parameters to launch an instance\. With launch templates, you can also create multiple versions of a template to make it faster and easier to launch new instances\. 

**To create a launch template from a launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Launch Configurations**\.

1. Select the launch configuration you want to copy and choose **Copy to launch template**\. This sets up a new launch template with the same name and options as the launch configuration you selected\.

1. For **New launch template name**, you can use the name of the launch configuration \(the default\) or type a new name\. Launch template names must be unique\.

1. \(Optional\) To create an Auto Scaling group using the new launch template, select **Create an Auto Scaling group using the new template**\. For more information, see [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

1. Choose **Submit**\.

After creating your launch template, you can update your existing Auto Scaling groups, as needed, with the launch template that you created\. For more information, see [Replacing a Launch Configuration with a Launch Template](replace-launch-config.md)\.