# Auto Scaling Groups<a name="AutoScalingGroup"></a>

An *Auto Scaling group* contains a collection of Amazon EC2 instances that share similar characteristics and are treated as a logical grouping for the purposes of instance scaling and management\. For example, if a single application operates across multiple instances, you might want to increase the number of instances in that group to improve the performance of the application\. Or, you can decrease the number of instances to reduce costs when demand is low\. Use the Auto Scaling group to scale the number of instances automatically based on criteria that you specify\. You could also maintain a fixed number of instances even if an instance becomes unhealthy\. This automatic scaling and maintaining the number of instances in an Auto Scaling group is the core functionality of the Amazon EC2 Auto Scaling service\.

An Auto Scaling group starts by launching enough instances to meet its desired capacity\. The Auto Scaling group maintains this number of instances by performing periodic health checks on the instances in the group\. If an instance becomes unhealthy, the group terminates the unhealthy instance and launches another instance to replace it\. For more information about health check replacements, see [Maintaining the Number of Instances in Your Auto Scaling Group](as-maintain-instance-levels.md)\.

You can use scaling policies to increase or decrease the number of instances in your group dynamically to meet changing conditions\. When the scaling policy is in effect, the Auto Scaling group adjusts the desired capacity of the group and launches or terminates the instances as needed\. You can also manually scale or scale on a schedule\. For more information, see [Scaling the Size of Your Auto Scaling Group](scaling_plan.md)\.

**Topics**
+ [Using Multiple Instance Types and Purchase Options](#asg-purchase-options)
+ [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)
+ [Creating an Auto Scaling Group Using a Launch Configuration](create-asg.md)
+ [Creating an Auto Scaling Group Using an EC2 Instance](create-asg-from-instance.md)
+ [Creating an Auto Scaling Group Using the Amazon EC2 Launch Wizard](create-asg-ec2-wizard.md)
+ [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)
+ [Using a Load Balancer With an Auto Scaling Group](autoscaling-load-balancer.md)
+ [Launching Spot Instances in Your Auto Scaling Group](asg-launch-spot-instances.md)
+ [Merging Your Auto Scaling Groups into a Single Multi\-Zone Group](merge-auto-scaling-groups.md)
+ [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)

## Using Multiple Instance Types and Purchase Options<a name="asg-purchase-options"></a>

To maximize your cost savings and obtain the desired scale for your application, you can provision and automatically scale Spot Instances with On\-Demand and Reserved Instances within a single Auto Scaling group\. 

Specify the common configuration parameters in a launch template and choose it when you create an Auto Scaling group\. When you configure the Auto Scaling group, specify the instance types, the instance distribution of On\-Demand Instances and Spot Instances, and other settings that tell the Auto Scaling group how to allocate instance types to fulfill capacity\. Deploying your application across multiple instance types is used to enhance availability, while giving you the benefit of the least expensive and available instance types\.

With Spot Instances, each instance type in each Availability Zone is a pool of spare compute capacity\. When you configure the Auto Scaling group, choose multiple Spot pools to allocate your Spot Instances across for high availability\. If you run a web service, we recommend specifying a high N \(number of Spot pools\), for example, N=10, to minimize the impact of Spot Instance interruptions if a pool in one of the Availability Zones becomes temporarily unavailable\. If you run batch processing or other non\-mission critical applications, you may specify a lower N, for example, N=2, to ensure that you provision Spot Instances from only the very lowest priced Spot pools available per Availability Zone\. For details on the pricing model for Spot Instances, see [Pricing and Savings](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html#spot-pricing) in the *Amazon EC2 User Guide for Linux Instances*\.

With On\-Demand Instances, you can choose what percentage of the group to launch as On\-Demand Instances and, optionally, a base number of On\-Demand Instances to start with\. If you choose to specify a base capacity of On\-Demand Instances, the Auto Scaling group ensures that this base capacity of On\-Demand Instances is provisioned first when the group scales out\. Anything beyond the base capacity uses percentages to determine how many On\-Demand Instances and Spot Instances to launch\. The behavior of the Auto Scaling group as it increases in size is as follows:


**Example: Scaling behavior**  

| Instances Distribution | Total Number of Running Instances Across Purchase Options | 
| --- | --- | 
|  | 10 | 20 | 30 | 40 | 
| Example 1 |  |  |  |  | 
| On\-Demand base: 10 | 10 | 10 | 10 | 10 | 
| On\-Demand percentage above base: 50% | 0 | 5 | 10 | 15 | 
| Spot percentage: 50% | 0 | 5 | 10 | 15 | 
| Example 2 |  |  |  |  | 
| On\-Demand base: 0 | 0 | 0 | 0 | 0 | 
| On\-Demand percentage above base: 0% | 0 | 0 | 0 | 0 | 
| Spot percentage: 100% | 10 | 20 | 30 | 40 | 
| Example 3 |  |  |  |  | 
| On\-Demand base: 0 | 0 | 0 | 0 | 0 | 
| On\-Demand percentage above base: 60% | 6 | 12 | 18 | 24 | 
| Spot percentage: 40% | 4 | 8 | 12 | 16 | 
| Example 4 |  |  |  |  | 
| On\-Demand base: 0 | 0 | 0 | 0 | 0 | 
| On\-Demand percentage above base: 100% | 10 | 20 | 30 | 40 | 
| Spot percentage: 0% | 0 | 0 | 0 | 0 | 
| Example 5 |  |  |  |  | 
| On\-Demand base: 12 | 10 | 12 | 12 | 12 | 
| On\-Demand percentage above base: 0% | 0 | 0 | 0 | 0 | 
| Spot percentage: 100% | 0 | 8 | 18 | 28 | 

For more information about how to configure a group to use multiple instance types and purchase options, see [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

For more information about Amazon EC2 pricing and purchase options, see [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/)\. 