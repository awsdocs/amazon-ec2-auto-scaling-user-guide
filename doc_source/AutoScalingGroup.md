# Auto Scaling Groups<a name="AutoScalingGroup"></a>

An *Auto Scaling group* contains a collection of Amazon EC2 instances that are treated as a logical grouping for the purposes of automatic scaling and management\. An Auto Scaling group also enables you to use Amazon EC2 Auto Scaling features such as health check replacements and scaling policies\. Both maintaining the number of instances in an Auto Scaling group and automatic scaling are the core functionality of the Amazon EC2 Auto Scaling service\.

The size of an Auto Scaling group depends on the number of instances that you set as the desired capacity\. You can adjust its size to meet demand, either manually or by using automatic scaling\. 

An Auto Scaling group starts by launching enough instances to meet its desired capacity\. It maintains this number of instances by performing periodic health checks on the instances in the group\. The Auto Scaling group continues to maintain a fixed number of instances even if an instance becomes unhealthy\. If an instance becomes unhealthy, the group terminates the unhealthy instance and launches another instance to replace it\. For more information, see [Health Checks for Auto Scaling Instances](healthcheck.md)\. 

You can use scaling policies to increase or decrease the number of instances in your group dynamically to meet changing conditions\. When the scaling policy is in effect, the Auto Scaling group adjusts the desired capacity of the group, between the minimum and maximum capacity values that you specify, and launches or terminates the instances as needed\. You can also scale on a schedule\. For more information, see [Scaling the Size of Your Auto Scaling Group](scaling_plan.md)\. 

An Auto Scaling group can launch On\-Demand Instances, Spot Instances, or both\. You can specify multiple purchase options for your Auto Scaling group only when you configure the group to use a launch template\. \(We recommend that you use launch templates instead of launch configurations to make sure that you can use the latest features of Amazon EC2\.\) 

Spot Instances provide you with access to unused Amazon EC2 capacity at steep discounts relative to On\-Demand prices\. For more information, see [Amazon EC2 Spot Instances](https://aws.amazon.com/ec2/spot/pricing/)\. There are key differences between Spot Instances and On\-Demand Instances:
+ The price for Spot Instances varies based on demand
+ Amazon EC2 can terminate an individual Spot Instance as the availability of, or price for, Spot Instances changes

When a Spot Instance is terminated, the Auto Scaling group attempts to launch a replacement instance to maintain the desired capacity for the group\. 

When instances are launched, if you specified multiple Availability Zones, the desired capacity is distributed across these Availability Zones\. If a scaling action occurs, Amazon EC2 Auto Scaling automatically maintains balance across all of the Availability Zones that you specify\.

This section provides detailed steps for creating an Auto Scaling group\. If you're new to Auto Scaling groups, start with [Getting Started with Amazon EC2 Auto Scaling](GettingStartedTutorial.md) to learn about the various building blocks that are used in Amazon EC2 Auto Scaling\. 

**Topics**
+ [Auto Scaling Groups with Multiple Instance Types and Purchase Options](asg-purchase-options.md)
+ [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)
+ [Creating an Auto Scaling Group Using a Launch Configuration](create-asg.md)
+ [Creating an Auto Scaling Group Using an EC2 Instance](create-asg-from-instance.md)
+ [Creating an Auto Scaling Group Using the Amazon EC2 Launch Wizard](create-asg-ec2-wizard.md)
+ [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)
+ [Elastic Load Balancing and Amazon EC2 Auto Scaling](autoscaling-load-balancer.md)
+ [Launching Spot Instances in Your Auto Scaling Group](asg-launch-spot-instances.md)
+ [Getting Recommendations for an Instance Type](asg-getting-recommendations.md)
+ [Replacing Auto Scaling Instances Based on Maximum Instance Lifetime](asg-max-instance-lifetime.md)
+ [Replacing Auto Scaling Instances Based on an Instance Refresh](asg-instance-refresh.md)
+ [Merging Your Auto Scaling Groups into a Single Multi\-Zone Group](merge-auto-scaling-groups.md)
+ [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)