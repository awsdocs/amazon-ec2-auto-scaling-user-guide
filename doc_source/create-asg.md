# Creating an Auto Scaling group using a launch configuration<a name="create-asg"></a>

When you create an Auto Scaling group, you must specify the necessary information to configure the Amazon EC2 instances, the subnets for the instances, and the initial number of instances\.

**Important**  
To configure the Amazon EC2 instances, you can specify a launch template, a launch configuration, or an EC2 instance\. We recommend that you use a launch template to make sure that you can use the latest features of Amazon EC2\. For more information, see [Launch templates](LaunchTemplates.md)\.

The following procedure demonstrates how to create an Auto Scaling group using a launch configuration\. You cannot modify a launch configuration after it is created, but you can replace the launch configuration for an Auto Scaling group\. For more information, see [Changing the launch configuration for an Auto Scaling group](change-launch-config.md)\. 

**Prerequisites**  
Create a launch configuration\. For more information, see [Creating a launch configuration](create-launch-config.md)\.

**To create an Auto Scaling group using a launch configuration \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. On the navigation bar at the top of the screen, choose the same Region that you used when you created the launch template\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group\.

1. To choose a launch configuration, do the following:

   1. For **Launch Template**, choose **Switch to launch configuration**\.

   1. For **Launch configuration**, choose an existing launch configuration\.

   1. Verify that your launch configuration supports all of the options that you are planning to use, and then choose **Next**\.

1. On the **Configure settings** page, under **Network**, for **VPC**, choose the VPC for the security groups that you specified in your launch configuration\. Launching instances using a combination of instance types and purchase options is not supported in EC2\-Classic\. 

1. For **Subnet**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information about high availability with Amazon EC2 Auto Scaling, see [Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. Choose **Next**\. 

   Or, you can accept the rest of the defaults, and choose **Skip to review**\. 

1. \(Optional\) On the **Configure advanced options** page, configure the following options, and then choose **Next**:

   1. To register your Amazon EC2 instances with a load balancer, choose an existing load balancer or create a new one\. For more information, see [Elastic Load Balancing and Amazon EC2 Auto Scaling](autoscaling-load-balancer.md)\. To create a new load balancer, follow the procedure in [Configure an Application Load Balancer or Network Load Balancer using the Amazon EC2 Auto Scaling console](attach-load-balancer-asg.md#as-create-load-balancer-console)\.

   1. To enable your Elastic Load Balancing \(`ELB`\) health checks, for **Health checks**, choose **ELB** under **Health check type**\. These health checks are optional when you enable load balancing\. 

   1. Under **Health check grace period**, enter the amount of time until Amazon EC2 Auto Scaling checks the health of instances after they are put into service\. The intention of this setting is to prevent Amazon EC2 Auto Scaling from marking instances as unhealthy and terminating them before they have time to come up\. The default is 300 seconds\.

1. \(Optional\) On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. For **Desired capacity**, enter the initial number of instances to launch\. When you change this number to a value outside of the minimum or maximum capacity limits, you must update the values of **Minimum capacity** or **Maximum capacity**\. For more information, see [Setting capacity limits for your Auto Scaling group](asg-capacity-limits.md)\.

   1. To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target Tracking Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

   1. Under **Instance scale\-in protection**, choose whether to enable instance scale\-in protection\. For more information, see [Instance scale\-in protection](as-instance-termination.md#instance-protection)\.

1. \(Optional\) To receive notifications, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Getting Amazon SNS notifications when your Auto Scaling group scales](ASGettingNotifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tagging Auto Scaling groups and instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

**To create an Auto Scaling group using the command line**

You can use one of the following commands:
+ [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)
+ [New\-ASAutoScalingGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)