# Launch Configurations<a name="LaunchConfiguration"></a>

A *launch configuration* is a template that an Auto Scaling group uses to launch EC2 instances\. When you create a launch configuration, you specify information for the instances such as the ID of the Amazon Machine Image \(AMI\), the instance type, a key pair, one or more security groups, and a block device mapping\. If you've launched an EC2 instance before, you specified the same information in order to launch the instance\.

**Important**  
When you create an Auto Scaling group, you must specify a launch template, launch configuration, or an EC2 instance\. We recommend that you use a launch template instead of a launch configuration\. For more information, see [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

You can specify your launch configuration with multiple Auto Scaling groups\. However, you can only specify one launch configuration for an Auto Scaling group at a time, and you can't modify a launch configuration after you've created it\. Therefore, if you want to change the launch configuration for an Auto Scaling group, you must create a launch configuration and then update your Auto Scaling group with the new launch configuration\.

**Topics**
+ [Creating a Launch Configuration](create-launch-config.md)
+ [Creating a Launch Configuration Using an EC2 Instance](create-lc-with-instanceID.md)
+ [Changing the Launch Configuration for an Auto Scaling Group](change-launch-config.md)
+ [Launching Auto Scaling Instances in a VPC](asg-in-vpc.md)