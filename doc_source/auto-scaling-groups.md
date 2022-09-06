# Auto Scaling groups<a name="auto-scaling-groups"></a>

An *Auto Scaling group* contains a collection of EC2 instances that are treated as a logical grouping for the purposes of automatic scaling and management\. An Auto Scaling group also enables you to use Amazon EC2 Auto Scaling features such as health check replacements and scaling policies\. Both maintaining the number of instances in an Auto Scaling group and automatic scaling are the core functionality of the Amazon EC2 Auto Scaling service\.

The size of an Auto Scaling group depends on the number of instances that you set as the desired capacity\. You can adjust its size to meet demand, either manually or by using automatic scaling\. 

An Auto Scaling group starts by launching enough instances to meet its desired capacity\. It maintains this number of instances by performing periodic health checks on the instances in the group\. The Auto Scaling group continues to maintain a fixed number of instances even if an instance becomes unhealthy\. If an instance becomes unhealthy, the group terminates the unhealthy instance and launches another instance to replace it\. For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\. 

You can use scaling policies to increase or decrease the number of instances in your group dynamically to meet changing conditions\. When the scaling policy is in effect, the Auto Scaling group adjusts the desired capacity of the group, between the minimum and maximum capacity values that you specify, and launches or terminates the instances as needed\. You can also scale on a schedule\. For more information, see [Scale the size of your Auto Scaling group](scale-your-group.md)\. 

An Auto Scaling group can launch On\-Demand Instances, Spot Instances, or both\. You can specify multiple purchase options for your Auto Scaling group only when you configure the group to use a launch template\. \(We recommend that you use launch templates instead of launch configurations to make sure that you can use the latest features of Amazon EC2\.\) 

Spot Instances provide you with access to unused EC2 capacity at steep discounts relative to On\-Demand prices\. For more information, see [Amazon EC2 Spot Instances](https://aws.amazon.com/ec2/spot/pricing/)\. There are key differences between Spot Instances and On\-Demand Instances:
+ The price for Spot Instances varies based on demand
+ Amazon EC2 can terminate an individual Spot Instance as the availability of, or price for, Spot Instances changes

When a Spot Instance is terminated, the Auto Scaling group attempts to launch a replacement instance to maintain the desired capacity for the group\. 

When instances are launched, if you specified multiple Availability Zones, the desired capacity is distributed across these Availability Zones\. If a scaling action occurs, Amazon EC2 Auto Scaling automatically maintains balance across all of the Availability Zones that you specify\.

If you're new to Auto Scaling groups, start by creating a launch template or a launch configuration and then use it to create an Auto Scaling group in which all instances have the same instance attributes\. You can set the following instance attributes by specifying them as part of the launch template or launch configuration: AMI, block devices, key pair, instance type, security groups, user data, EC2 instance monitoring, instance profile, kernel, ramdisk, the tenancy of the instance, whether the instance has a public IP address, and whether the instance is EBS\-optimized\. The [Get started with Amazon EC2 Auto Scaling](get-started-with-ec2-auto-scaling.md) tutorial provides a quick introduction to the various building blocks that are used in Amazon EC2 Auto Scaling\.

**Topics**
+ [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)
+ [Create an Auto Scaling group using attribute\-based instance type selection](create-asg-instance-type-requirements.md)
+ [Create an Auto Scaling group using a launch template](create-asg-launch-template.md)
+ [Create an Auto Scaling group using a launch configuration](create-asg-launch-configuration.md)
+ [Create an Auto Scaling group using parameters from an existing instance](create-asg-from-instance.md)
+ [Create an Auto Scaling group using the Amazon EC2 launch wizard](create-asg-ec2-wizard.md)
+ [Tag Auto Scaling groups and instances](ec2-auto-scaling-tagging.md)
+ [Replace Auto Scaling instances](ec2-auto-scaling-group-replacing-instances.md)
+ [Delete your Auto Scaling infrastructure](as-process-shutdown.md)