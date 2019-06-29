# Creating an Auto Scaling Group Using a Launch Configuration<a name="create-asg"></a>

When you create an Auto Scaling group, you must specify the necessary information to configure the Amazon EC2 instances, the subnets for the instances, and the initial number of instances\.

**Important**  
To configure the Amazon EC2 instances, you can specify a launch template, a launch configuration, or an EC2 instance\. We recommend that you use a launch template to make sure that you can use the latest features of Amazon EC2\. For more information, see [Launch Templates](LaunchTemplates.md)\.

The following procedure demonstrates how to create an Auto Scaling group using a launch configuration\. You cannot modify a launch configuration after it is created, but you can replace the launch configuration for an Auto Scaling group\. For more information, see [Changing the Launch Configuration for an Auto Scaling Group](change-launch-config.md)\. 

**Prerequisites**  
Create a launch configuration\. For more information, see [Creating a Launch Configuration](create-launch-config.md)\.

**To create an Auto Scaling group using a launch configuration \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch configuration\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration**, choose an existing launch configuration, and then choose **Next Step**\.
**Note**  
If you do not have any launch configurations, you're first prompted to create one before you can continue with the steps to create an Auto Scaling group\.

1. On the **Configure Auto Scaling group details** page, do the following:

   1. For **Group name**, enter a name for your Auto Scaling group\.

   1. For **Group size**, enter the initial number of instances for your Auto Scaling group\.

   1. For **Network**, choose a VPC for your Auto Scaling group\.

   1. For **Subnet**, choose one or more subnets in your VPC\. Use subnets in multiple Availability Zones for high availability\. For more information about high availability with Amazon EC2 Auto Scaling, see [Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

   1. \(Optional\) To register your Amazon EC2 instances with a load balancer, choose **Advanced Details**, choose **Receive traffic from one or more load balancers**, and choose one or more Classic Load Balancers or target groups\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, choose one of the following options, and then choose **Next: Configure Notifications**:
   + To manually adjust the size of the Auto Scaling group as needed, choose **Keep this group at its initial size**\. For more information, see [Manual Scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.
   + To automatically adjust the size of the Auto Scaling group based on criteria that you specify, choose **Use scaling policies to adjust the capacity of this group** and follow the directions\. For more information, see [Configure Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

1. \(Optional\) To receive notifications, choose **Add notification**, configure the notification, and then choose **Next: Configure Tags**\.

1. \(Optional\) To add tags, choose **Edit tags**, provide a tag key and value for each tag, and then choose **Review**\.

   Alternatively, you can add tags later on\. For more information, see [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

**To create an Auto Scaling group using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)