# Creating an Auto Scaling Group Using a Launch Configuration<a name="create-asg"></a>

When you create an Auto Scaling group, you must specify the information needed to configure the Auto Scaling instances and the minimum number of instances your group must maintain at all times\.

**Important**  
To configure the Auto Scaling instances, you can specify a launch template, a launch configuration, or an EC2 instance\. We recommend that you use a launch template to ensure that you can use the latest features of Amazon EC2\. For more information, see [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

The following procedures demonstrate how to create an Auto Scaling group using a launch configuration\. You cannot modify a launch configuration after it is created, but you can replace the launch configuration for an Auto Scaling group\. For more information, see [Changing the Launch Configuration for an Auto Scaling Group](change-launch-config.md)\.

**Prerequisites**  
Create a launch configuration\. For more information, see [Creating a Launch Configuration](create-launch-config.md)\.

**To create an Auto Scaling group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the same region that you used when you created the launch configuration\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, select **Create an Auto Scaling group from an existing launch configuration**, select a launch configuration, and then choose **Next Step**\.
**Note**  
If you do not have any launch configurations, you're first prompted to create one before you can continue with the steps to create an Auto Scaling group\.

1. On the **Configure Auto Scaling group details** page, do the following:

   1. \(Optional\) To use a version of the launch template other than the default, select the version from **Launch template version**\.

   1. For **Group name**, type a name for your Auto Scaling group\.

   1. For **Group size**, type the initial number of instances for your Auto Scaling group\.

   1. For **Network**, select a VPC for your Auto Scaling group\.

   1. For **Subnet**, select one or more subnets\.

   1. \(Optional\) To register your Auto Scaling instances with a load balancer, select **Receive traffic from one or more load balancers** and select one or more Classic Load Balancers or target groups\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, select one of the following options, and then choose **Next: Configure Notifications**:

   + To manually adjust the size of the Auto Scaling group as needed, select **Keep this group at its initial size**\. For more information, see [Manual Scaling](as-manual-scaling.md)\.

   + To automatically adjust the size of the Auto Scaling group based on criteria that you specify, select **Use scaling policies to adjust the capacity of this group** and follow the directions\. For more information, see Configure Scaling Policies\.

1. \(Optional\) To receive notifications, choose **Add notification**, configure the notification, and then choose **Next: Configure Tags**\.

1. \(Optional\) To add tags, choose **Edit tags**, provide a tag key and value for each tag, and then choose **Review**\.

   Alternatively, you can add tags later on\. For more information, see [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

**To create an Auto Scaling group using the command line**

You can use one of the following commands:

+ [create\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)

+ [New\-ASAutoScalingGroup](http://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)