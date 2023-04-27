# Launch configurations<a name="launch-configurations"></a>

**Important**  
We strongly recommend that you do not use launch configurations\. They do not provide full functionality for Amazon EC2 Auto Scaling or Amazon EC2\.   
Launch configurations no longer add support for new Amazon EC2 instance types that are released after **December 31, 2022**\. In addition, any new accounts created after **March 31, 2023** will not have the option to create new launch configurations through the console\. However, API and CLI access will be available to new accounts created between **March 31, 2023** and **December 31, 2023** to support customers with automation use cases\. New accounts created after **December 31, 2023** will not be able to create new launch configurations by using the console, API, or CLI\. For information about migrating existing Auto Scaling groups to launch templates, see [Migrate to launch templates](launch-templates.md#migrate-to-launch-templates) and [Amazon EC2 Auto Scaling will no longer add support for new EC2 features to Launch Configurations](http://aws.amazon.com/blogs/compute/amazon-ec2-auto-scaling-will-no-longer-add-support-for-new-ec2-features-to-launch-configurations/) on the AWS Compute Blog\.

A *launch configuration* is an instance configuration template that an Auto Scaling group uses to launch EC2 instances\. When you create a launch configuration, you specify information for the instances\. Include the ID of the Amazon Machine Image \(AMI\), the instance type, a key pair, one or more security groups, and a block device mapping\. If you've launched an EC2 instance before, you specified the same information in order to launch the instance\.

You can specify your launch configuration with multiple Auto Scaling groups\. However, you can only specify one launch configuration for an Auto Scaling group at a time, and you can't modify a launch configuration after you've created it\. To change the launch configuration for an Auto Scaling group, you must create a launch configuration and then update your Auto Scaling group with it\.

**Topics**
+ [Create a launch configuration](create-launch-config.md)
+ [Create a launch configuration using an EC2 instance](create-lc-with-instanceID.md)
+ [Change the launch configuration for an Auto Scaling group](change-launch-config.md)
+ [Configure instance tenancy with a launch configuration](auto-scaling-dedicated-instances.md)