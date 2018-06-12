# Launch Configurations<a name="LaunchConfiguration"></a>

A *launch configuration* is a template that an Auto Scaling group uses to launch EC2 instances\. When you create a launch configuration, you specify information for the instances such as the ID of the Amazon Machine Image \(AMI\), the instance type, a key pair, one or more security groups, and a block device mapping\. If you've launched an EC2 instance before, you specified the same information in order to launch the instance\.

You can specify your launch configuration with multiple Auto Scaling groups\. However, you can only specify one launch configuration for an Auto Scaling group at a time, and you can't modify a launch configuration after you've created it\. Therefore, if you want to change the launch configuration for an Auto Scaling group, you must create a launch configuration and then update your Auto Scaling group with the new launch configuration\.

Keep in mind that whenever you create an Auto Scaling group, you must specify a launch configuration, a launch template, or an EC2 instance\. When you create an Auto Scaling group using an EC2 instance, Amazon EC2 Auto Scaling automatically creates a launch configuration for you and associates it with the Auto Scaling group\. For more information, see [Creating an Auto Scaling Group Using an EC2 Instance](create-asg-from-instance.md)\. Alternatively, if you create a launch template, you can use your launch template to create an Auto Scaling group instead of creating a launch configuration\. For more information, see [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

**Topics**
+ [Creating a Launch Configuration](create-launch-config.md)
+ [Creating a Launch Configuration Using an EC2 Instance](create-lc-with-instanceID.md)
+ [Changing the Launch Configuration for an Auto Scaling Group](change-launch-config.md)
+ [Copying a Launch Configuration to a Launch Template](copy-launch-config.md)
+ [Replacing a Launch Configuration with a Launch Template](replace-launch-config.md)
+ [Launching Auto Scaling Instances in a VPC](asg-in-vpc.md)