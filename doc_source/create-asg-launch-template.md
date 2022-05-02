# Create an Auto Scaling group using a launch template<a name="create-asg-launch-template"></a>

When you create an Auto Scaling group, you must specify the necessary information to configure the Amazon EC2 instances, the Availability Zones and VPC subnets for the instances, the desired capacity, and the minimum and maximum capacity limits\. 

To configure Amazon EC2 instances that are launched by your Auto Scaling group, you can specify a launch template or a launch configuration\. The following procedure demonstrates how to create an Auto Scaling group using a launch template\. 

To update the configuration of the EC2 instances after the group is created, you can create a new version of the launch template\. After you change the launch template for an Auto Scaling group, any new instances are launched using the new configuration options, but existing instances are not affected\. To update the existing instances, terminate them so that they are replaced by your Auto Scaling group, or allow auto scaling to gradually replace older instances with newer instances based on your [termination policies](as-instance-termination.md)\.

**Note**  
You can also replace all instances in the Auto Scaling group to launch new instances that use the new launch template\. For more information, see [Replace Auto Scaling instances](ec2-auto-scaling-group-replacing-instances.md)\.

**Prerequisites**
+ You must have created a launch template\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\.
+ You must have IAM permissions to create an Auto Scaling group using a launch template\. Your `ec2:RunInstances` permissions are checked when use a launch template\. Your `iam:PassRole` permissions are also checked if the launch template specifies an IAM role\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

**To create an Auto Scaling group using a launch template \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch template\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, do the following:

   1. For **Auto Scaling group name**, enter a name for your Auto Scaling group\.

   1. For **Launch template**, choose an existing launch template\.

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\. 

   1. Verify that your launch template supports all of the options that you are planning to use, and then choose **Next**\.

1. On the **Choose instance launch options** page, under **Network**, for **VPC**, choose a VPC\. The Auto Scaling group must be created in the same VPC as the security group you specified in your launch template\.

1. For **Availability Zones and subnets**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information, see [Considerations when choosing VPC subnets](asg-in-vpc.md#as-vpc-considerations)\.

1. If you created a launch template with an instance type specified, then you can continue to the next step to create an Auto Scaling group that uses the instance type in the launch template\. 

   Alternatively, you can choose the **Override launch template** option if no instance type is specified in your launch template or if you want to use multiple instance types for auto scaling\. For more information, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.

1. Choose **Next** to continue to the next step\. 

   Or, you can accept the rest of the defaults, and choose **Skip to review**\. 

1. \(Optional\) On the **Configure advanced options** page, configure the following options, and then choose **Next**:

   1. To register your Amazon EC2 instances with a load balancer, choose an existing load balancer or create a new one\. For more information, see [Use Elastic Load Balancing to distribute traffic across the instances in your Auto Scaling group](autoscaling-load-balancer.md)\. To create a new load balancer, follow the procedure in [Configure an Application Load Balancer or Network Load Balancer from the Amazon EC2 Auto Scaling console](as-create-load-balancer-console.md)\.

   1. To enable your Elastic Load Balancing \(`ELB`\) health checks, for **Health checks**, choose **ELB** under **Health check type**\. These health checks are optional when you enable load balancing\. 

   1. Under **Health check grace period**, enter the amount of time until Amazon EC2 Auto Scaling checks the Elastic Load Balancing health status of an instance after it enters the `InService` state\. For more information, see [Health check grace period](ec2-auto-scaling-health-checks.md#health-check-grace-period)\.

   1. Under **Additional settings**, **Monitoring**, choose whether to enable CloudWatch group metrics collection\. These metrics provide measurements that can be indicators of a potential issue, such as number of terminating instances or number of pending instances\. For more information, see [Monitor CloudWatch metrics for your Auto Scaling groups and instances](ec2-auto-scaling-cloudwatch-monitoring.md)\.

   1. For **Enable default instance warmup**, select this option and choose the warm\-up time for your application\. If you are creating an Auto Scaling group that has a scaling policy, the default instance warmup feature improves the Amazon CloudWatch metrics used for dynamic scaling\. For more information, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.

1. \(Optional\) On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. For **Desired capacity**, enter the initial number of instances to launch\. When you change this number to a value outside of the minimum or maximum capacity limits, you must update the values of **Minimum capacity** or **Maximum capacity**\. For more information, see [Set capacity limits on your Auto Scaling group](asg-capacity-limits.md)\.

   1. To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target Tracking Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

   1. Under **Instance scale\-in protection**, choose whether to enable instance scale\-in protection\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

1. \(Optional\) To receive notifications, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Get Amazon SNS notifications when your Auto Scaling group scales](ec2-auto-scaling-sns-notifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tag Auto Scaling groups and instances](ec2-auto-scaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

**To create an Auto Scaling group using the command line**

You can use one of the following commands:
+ [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)
+ [New\-ASAutoScalingGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)