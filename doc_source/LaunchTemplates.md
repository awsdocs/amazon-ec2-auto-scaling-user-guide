# Launch templates<a name="LaunchTemplates"></a>

A launch template is similar to a [launch configuration](LaunchConfiguration.md), in that it specifies instance configuration information\. It includes the ID of the Amazon Machine Image \(AMI\), the instance type, a key pair, security groups, and other parameters used to launch EC2 instances\. However, defining a launch template instead of a launch configuration allows you to have multiple versions of a launch template\. 

With versioning of launch templates, you can create a subset of the full set of parameters\. Then, you can reuse it to create other versions of the same launch template\. For example, you can create a launch template that defines a base configuration without an AMI or user data script\. After you create your launch template, you can create a new version and add the AMI and user data that has the latest version of your application for testing\. This results in two versions of the launch template\. Storing a base configuration helps you to maintain the required general configuration parameters\. You can create a new version of your launch template from the base configuration whenever you want\. You can also delete the versions used for testing your application when you no longer need them\. 

We recommend that you use launch templates to ensure that you're accessing the latest features and improvements\. Not all Amazon EC2 Auto Scaling features are available when you use launch configurations\. For example, you cannot create an Auto Scaling group that launches both Spot and On\-Demand Instances or that specifies multiple instance types\. You must use a launch template to configure these features\. For more information, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.

With launch templates, you can also use newer features of Amazon EC2\. This includes the current generation of EBS Provisioned IOPS volumes \(io2\), EBS volume tagging, [T2 Unlimited instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-unlimited-mode-concepts.html), Elastic Inference, and [Dedicated Hosts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html), to name a few\. Dedicated Hosts are physical servers with EC2 instance capacity that are dedicated to your use\. While Amazon EC2 [Dedicated Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html) also run on dedicated hardware, the advantage of using Dedicated Hosts over Dedicated Instances is that you can bring eligible software licenses from external vendors and use them on EC2 instances\. 

If you currently use launch configurations, you can migrate data from your existing launch configurations to launch templates by [copying them in the console](https://docs.aws.amazon.com/autoscaling/ec2/userguide/copy-launch-config.html)\. Then, you can migrate your deployed Auto Scaling groups that use a launch configuration to a new launch template\. To do this, start an instance refresh to do a rolling update of your group\. For more information, see [Replacing Auto Scaling instances](ec2-auto-scaling-group-replacing-instances.md)\. 

When you create a launch template, all parameters are optional\. However, if a launch template does not specify an AMI, you cannot add the AMI when you create your Auto Scaling group\. If you specify an AMI but no instance type, you can add one or more instance types when you create your Auto Scaling group\.

**Topics**
+ [Permissions](#launch-templates-permissions)
+ [Creating a launch template for an Auto Scaling group](create-launch-template.md)
+ [Copy launch configurations to launch templates](copy-launch-config.md)
+ [Replacing a launch configuration with a launch template](replace-launch-config.md)
+ [Examples for creating and managing launch templates with the AWS Command Line Interface \(AWS CLI\)](examples-launch-templates-aws-cli.md)

## Permissions<a name="launch-templates-permissions"></a>

The procedures in this section assume that you already have the required permissions to use launch templates\. With permissions in place, you can create and manage launch templates\. You can also create and update Auto Scaling groups and specify a launch template instead of a launch configuration\. 

When you update or create an Auto Scaling group and specify a launch template, your `ec2:RunInstances` permissions are checked\. If you do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. 

Some additional functionality in the request requires additional permissions, such as the ability to pass an IAM role to provisioned instances or to add tags to provisioned instances and volumes\.

For information about how an administrator grants you permissions, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.