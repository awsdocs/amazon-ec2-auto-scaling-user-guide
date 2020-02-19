# Launch Templates<a name="LaunchTemplates"></a>

A launch template is similar to a [launch configuration](LaunchConfiguration.md), in that it specifies instance configuration information\. Included are the ID of the Amazon Machine Image \(AMI\), the instance type, a key pair, security groups, and the other parameters that you use to launch EC2 instances\. However, defining a launch template instead of a launch configuration allows you to have multiple versions of a template\. With versioning, you can create a subset of the full set of parameters and then reuse it to create other templates or template versions\. For example, you can create a default template that defines common configuration parameters such as tags or network configurations, and allow the other parameters to be specified as part of another version of the same template\. 

We recommend that you use launch templates instead of launch configurations to ensure that you can use the latest features of Amazon EC2\. 

With launch templates, you can also provision capacity across multiple instance types using both On\-Demand Instances and Spot Instances to achieve the desired scale, performance, and cost\. For more information, see [Auto Scaling Groups with Multiple Instance Types and Purchase Options](asg-purchase-options.md)\.

If you currently use launch configurations, you can specify a launch template when you update an Auto Scaling group that was created using a launch configuration\.

To create a launch template to use with an Auto Scaling group, create the template from scratch, create a new version of an existing template, or copy the parameters from a launch configuration, running instance, or other template\. 

The following topics describe the most common procedures for creating and working with launch templates for use with your Auto Scaling groups\. For more information about launch templates, see the [launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html) section of the *Amazon EC2 User Guide for Linux Instances*\. 

**Topics**
+ [Creating a Launch Template for an Auto Scaling Group](create-launch-template.md)
+ [Copying a Launch Configuration to a Launch Template](copy-launch-config.md)
+ [Replacing a Launch Configuration with a Launch Template](replace-launch-config.md)