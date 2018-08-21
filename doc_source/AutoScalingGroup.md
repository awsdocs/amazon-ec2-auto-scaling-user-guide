# Auto Scaling Groups<a name="AutoScalingGroup"></a>

An *Auto Scaling group* contains a collection of EC2 instances that share similar characteristics and are treated as a logical grouping for the purposes of instance scaling and management\. For example, if a single application operates across multiple instances, you might want to increase the number of instances in that group to improve the performance of the application, or decrease the number of instances to reduce costs when demand is low\. You can use the Auto Scaling group to scale the number of instances automatically based on criteria that you specify, or maintain a fixed number of instances even if an instance becomes unhealthy\. This automatic scaling and maintaining the number of instances in an Auto Scaling group is the core functionality of the Amazon EC2 Auto Scaling service\.

An Auto Scaling group starts by launching enough EC2 instances to meet its desired capacity\. The Auto Scaling group maintains this number of instances by performing periodic health checks on the instances in the group\. If an instance becomes unhealthy, the group terminates the unhealthy instance and launches another instance to replace it\. For more information about health check replacements, see [Maintaining the Number of Instances in Your Auto Scaling Group](as-maintain-instance-levels.md)\.

You can use scaling policies to increase or decrease the number of running EC2 instances in your group dynamically to meet changing conditions\. When the scaling policy is in effect, the Auto Scaling group adjusts the desired capacity of the group and launches or terminates the instances as needed\. If you manually scale or scale on a schedule, you must adjust the desired capacity of the group in order for the changes to take effect\. For more information, see [Scaling the Size of Your Auto Scaling Group](scaling_plan.md)\.

Before you get started, take the time to review your application thoroughly as it runs in the AWS cloud\. Take note of the following:
+ How long it takes to launch and configure a server
+ What metrics have the most relevance to your application's performance
+ How many Availability Zones you want the Auto Scaling group to span
+ Do you want to scale to increase or decrease capacity? Do you just want to ensure that a specific number of servers are always running? \(Keep in mind that Amazon EC2 Auto Scaling can do both simultaneously\.\)
+ What existing resources \(such as EC2 instances or AMIs\) you can use

The better you understand your application, the more effective you can make your Auto Scaling architecture\.

**Topics**
+ [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)
+ [Creating an Auto Scaling Group Using a Launch Configuration](create-asg.md)
+ [Creating an Auto Scaling Group Using an EC2 Instance](create-asg-from-instance.md)
+ [Creating an Auto Scaling Group Using the Amazon EC2 Launch Wizard](create-asg-ec2-wizard.md)
+ [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)
+ [Using a Load Balancer With an Auto Scaling Group](autoscaling-load-balancer.md)
+ [Launching Spot Instances in Your Auto Scaling Group](asg-launch-spot-instances.md)
+ [Merging Your Auto Scaling Groups into a Single Multi\-Zone Group](merge-auto-scaling-groups.md)
+ [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)