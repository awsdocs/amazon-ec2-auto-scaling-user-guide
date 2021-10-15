# Creating an Auto Scaling group using a launch template<a name="create-asg-launch-template"></a>

To configure Amazon EC2 instances that are launched by your Auto Scaling group, you can specify a launch template, a launch configuration, or an EC2 instance\. The following procedure demonstrates how to create an Auto Scaling group using a launch template\. 

With launch templates, you can configure the Auto Scaling group to dynamically choose either the default version or the latest version of the launch template when a scale\-out event occurs\. For example, you configure your Auto Scaling group to choose the current default version of a launch template\. To change the configuration of the EC2 instances to be launched by the group, create or designate a new default version of the launch template\. Alternatively, you can choose the specific version of the launch template that the group uses to launch EC2 instances\. You can change these selections anytime by updating the group\. 

Each launch template includes the information that Amazon EC2 needs to launch instances, such as an AMI and instance type\. You can create an Auto Scaling group that adheres to the launch template\. Or, you can override the instance type in the launch template and combine On\-Demand and Spot Instances\. For more information, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\. 

The Auto Scaling group specifies the desired capacity and additional information that Amazon EC2 needs to launch instances, such as the Availability Zones and VPC subnets\. You can set capacity to a fixed number of instances, or you can take advantage of automatic scaling to adjust capacity based on actual demand\. 

**Prerequisites**
+ You must have created a launch template that includes the parameters required to launch an EC2 instance\. For information about these parameters and the limitations that apply when creating a launch template for use with an Auto Scaling group, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.
+ You must have IAM permissions to create an Auto Scaling group using a launch template and also to create EC2 resources for the instances\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

**To create an Auto Scaling group using a launch template \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. On the navigation bar at the top of the screen, choose the same Region that you used when you created the launch template\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, do the following:

   1. For **Auto Scaling group name**, enter a name for your Auto Scaling group\.

   1. For **Launch Template**, choose an existing launch template\.

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\. 

   1. Verify that your launch template supports all of the options that you are planning to use, and then choose **Next**\.

1. On the **Configure settings** page, for **Purchase options and instance types**, choose **Adhere to the launch template** to use the EC2 instance type and purchase option that are specified in the launch template\. 

1. Under **Network**, for **VPC**, choose the VPC for the security groups that you specified in your launch template\.

1. For **Subnet**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information about high availability with Amazon EC2 Auto Scaling, see [Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. Choose **Next**\. 

   Or, you can accept the rest of the defaults, and choose **Skip to review**\. 

1. \(Optional\) On the **Configure advanced options** page, configure the following options, and then choose **Next**:

   1. To register your Amazon EC2 instances with a load balancer, choose an existing load balancer or create a new one\. For more information, see [Elastic Load Balancing and Amazon EC2 Auto Scaling](autoscaling-load-balancer.md)\. To create a new load balancer, follow the procedure in [Configure an Application Load Balancer or Network Load Balancer using the Amazon EC2 Auto Scaling console](attach-load-balancer-asg.md#as-create-load-balancer-console)\.

   1. To enable your Elastic Load Balancing \(`ELB`\) health checks, for **Health checks**, choose **ELB** under **Health check type**\. These health checks are optional when you enable load balancing\. 

   1. Under **Health check grace period**, enter the amount of time until Amazon EC2 Auto Scaling checks the health of instances after they are put into service\. The intention of this setting is to prevent Amazon EC2 Auto Scaling from marking instances as unhealthy and terminating them before they have time to come up\. The default is 300 seconds\.

1. \(Optional\) On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. For **Desired capacity**, enter the initial number of instances to launch\. When you change this number to a value outside of the minimum or maximum capacity limits, you must update the values of **Minimum capacity** or **Maximum capacity**\. For more information, see [Setting capacity limits on your Auto Scaling group](asg-capacity-limits.md)\.

   1. To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target Tracking Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

   1. Under **Instance scale\-in protection**, choose whether to enable instance scale\-in protection\. For more information, see [Using instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

1. \(Optional\) To receive notifications, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Getting Amazon SNS notifications when your Auto Scaling group scales](ASGettingNotifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tagging Auto Scaling groups and instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

**To create an Auto Scaling group using the command line**

You can use one of the following commands:
+ [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)
+ [New\-ASAutoScalingGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)