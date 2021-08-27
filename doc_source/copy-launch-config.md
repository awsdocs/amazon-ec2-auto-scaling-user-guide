# Copy launch configurations to launch templates<a name="copy-launch-config"></a>

To migrate from launch configurations to launch templates, you must copy or recreate your launch configurations as launch templates\. We recommend that you migrate to launch templates to take advantage of the latest features of Amazon EC2 and Amazon EC2 Auto Scaling\. 

If you copy your launch configurations, you can migrate them all at once, or you can perform an incremental migration over time by choosing which launch configurations to copy\. The copying feature is available only from the console\. 

**To copy a launch configuration to a launch template \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\.

1. Select the launch configuration you want to copy and choose **Copy to launch template, Copy selected**\. This sets up a new launch template with the same name and options as the launch configuration that you selected\.

1. For **New launch template name**, you can use the name of the launch configuration \(the default\) or enter a new name\. Launch template names must be unique\.

1. \(Optional\) To create an Auto Scaling group using the new launch template, select **Create an Auto Scaling group using the new template**\.

1. Choose **Copy**\.

If you know that you want to copy all launch configurations to launch templates, use the following procedure\.

**To copy all launch configurations to launch templates \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\.

1. Choose **Copy to launch template, Copy all**\. This copies each launch configuration in the current Region to a new launch template with the same name and options\.

1. Choose **Copy**\.

Next, you can update your existing Auto Scaling groups to specify the launch templates that you created\. For more information, see [Replacing a launch configuration with a launch template](replace-launch-config.md)\. As another option, you can follow the procedure in [Replacing Auto Scaling instances based on an instance refresh](asg-instance-refresh.md) to add the new launch templates to your Auto Scaling groups and update your Auto Scaling instances immediately\.