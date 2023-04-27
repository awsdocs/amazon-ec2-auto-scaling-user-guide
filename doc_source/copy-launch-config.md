# Copy launch configurations to launch templates<a name="copy-launch-config"></a>

To migrate your existing Auto Scaling groups from launch configurations to launch templates, you must copy or recreate your launch configurations as launch templates\. This topic shows you how to copy your launch configurations to launch templates using the Amazon EC2 Auto Scaling console\. The copying feature is available only from the console\. 

**To copy a launch configuration to a launch template \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Launch Configurations**\.

1. Select the launch configuration you want to copy and choose **Copy to launch template, Copy selected**\. This sets up a new launch template with the same name and options as the launch configuration that you selected\.

1. For **New launch template name**, you can use the name of the launch configuration \(the default\) or enter a new name\. Launch template names must be unique\.

1. \(Optional\) To create an Auto Scaling group using the new launch template, select **Create an Auto Scaling group using the new template**\.

1. Choose **Copy**\.

If you know that you want to copy all launch configurations to launch templates, use the following procedure\.

**To copy all launch configurations to launch templates \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Launch Configurations**\.

1. Choose **Copy to launch template, Copy all**\. This copies each launch configuration in the current Region to a new launch template with the same name and options\.

1. Choose **Copy**\.

**Note**  
Next, you can update your existing Auto Scaling groups and choose the launch templates that you created\. For more information, see the following topics:  
To switch an Auto Scaling group to use a launch template, see [Replace a launch configuration with a launch template](replace-launch-config.md)\.
To migrate all your Auto Scaling groups to use launch templates, see [Migrate to launch templates](launch-templates.md#migrate-to-launch-templates)\.