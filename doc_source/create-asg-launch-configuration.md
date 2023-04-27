# Create an Auto Scaling group using a launch configuration<a name="create-asg-launch-configuration"></a>

**Important**  
We strongly recommend that you do not use launch configurations\. They do not provide full functionality for Amazon EC2 Auto Scaling or Amazon EC2\.   
Launch configurations no longer add support for new Amazon EC2 instance types that are released after **December 31, 2022**\. In addition, any new accounts created after **March 31, 2023** will not have the option to create new launch configurations through the console\. However, API and CLI access will be available to new accounts created between **March 31, 2023** and **December 31, 2023** to support customers with automation use cases\. New accounts created after **December 31, 2023** will not be able to create new launch configurations by using the console, API, or CLI\. For information about migrating existing Auto Scaling groups to launch templates, see [Migrate to launch templates](launch-templates.md#migrate-to-launch-templates) and [Amazon EC2 Auto Scaling will no longer add support for new EC2 features to Launch Configurations](http://aws.amazon.com/blogs/compute/amazon-ec2-auto-scaling-will-no-longer-add-support-for-new-ec2-features-to-launch-configurations/) on the AWS Compute Blog\.

When you create an Auto Scaling group, you must specify the necessary information to configure the Amazon EC2 instances, the Availability Zones and VPC subnets for the instances, the desired capacity, and the minimum and maximum capacity limits\.

The following procedure demonstrates how to create an Auto Scaling group using a launch configuration\. You cannot modify a launch configuration after it is created, but you can replace the launch configuration for an Auto Scaling group\. For more information, see [Change the launch configuration for an Auto Scaling group](change-launch-config.md)\. 

**Prerequisites**  
You must have created a launch configuration\. For more information, see [Create a launch configuration](create-launch-config.md)\.

**To create an Auto Scaling group using a launch configuration \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch configuration\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group\.

1. To choose a launch configuration, do the following:

   1. For **Launch template**, choose **Switch to launch configuration**\.

   1. For **Launch configuration**, choose an existing launch configuration\.

   1. Verify that your launch configuration supports all of the options that you are planning to use, and then choose **Next**\.

1. On the **Configure instance launch options** page, under **Network**, for **VPC**, choose a VPC\. The Auto Scaling group must be created in the same VPC as the security group you specified in your launch configuration\.

1. For **Availability Zones and subnets**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information, see [Considerations when choosing VPC subnets](asg-in-vpc.md#as-vpc-considerations)\.

1. Choose **Next**\. 

   Or, you can accept the rest of the defaults, and choose **Skip to review**\. 

1. \(Optional\) On the **Configure advanced options** page, configure the following options, and then choose **Next**:

   1. To register your Amazon EC2 instances with a load balancer, choose an existing load balancer or create a new one\. For more information, see [Use Elastic Load Balancing to distribute traffic across the instances in your Auto Scaling group](autoscaling-load-balancer.md)\. To create a new load balancer, follow the procedure in [Configure an Application Load Balancer or Network Load Balancer from the Amazon EC2 Auto Scaling console](as-create-load-balancer-console.md)\.

   1. \(Optional\) For **Health checks**, **Additional health check types**, select **Turn on Elastic Load Balancing health checks**\.

   1. \(Optional\) For **Health check grace period**, enter the amount of time, in seconds\. This is how long Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance after it enters the `InService` state\. For more information, see [Set the health check grace period for an Auto Scaling group](health-check-grace-period.md)\. 

   1. Under **Additional settings**, **Monitoring**, choose whether to enable CloudWatch group metrics collection\. These metrics provide measurements that can be indicators of a potential issue, such as number of terminating instances or number of pending instances\. For more information, see [Monitor CloudWatch metrics for your Auto Scaling groups and instances](ec2-auto-scaling-cloudwatch-monitoring.md)\.

   1. For **Enable default instance warmup**, select this option and choose the warm\-up time for your application\. If you are creating an Auto Scaling group that has a scaling policy, the default instance warmup feature improves the Amazon CloudWatch metrics used for dynamic scaling\. For more information, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.

1. \(Optional\) On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. For **Desired capacity**, enter the initial number of instances to launch\. When you change this number to a value outside of the minimum or maximum capacity limits, you must update the values of **Minimum capacity** or **Maximum capacity**\. For more information, see [Set capacity limits on your Auto Scaling group](asg-capacity-limits.md)\.

   1. To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target tracking scaling policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)\.

   1. Under **Instance scale\-in protection**, choose whether to enable instance scale\-in protection\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

1. \(Optional\) To receive notifications, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Get Amazon SNS notifications when your Auto Scaling group scales](ec2-auto-scaling-sns-notifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tag Auto Scaling groups and instances](ec2-auto-scaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

**To create an Auto Scaling group using the command line**

You can use one of the following commands:
+ [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)
+ [New\-ASAutoScalingGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)