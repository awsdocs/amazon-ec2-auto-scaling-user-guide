# What is Amazon EC2 Auto Scaling?<a name="what-is-amazon-ec2-auto-scaling"></a>

Amazon EC2 Auto Scaling helps you ensure that you have the correct number of Amazon EC2 instances available to handle the load for your application\. You create collections of EC2 instances, called *Auto Scaling groups*\. You can specify the minimum number of instances in each Auto Scaling group, and Amazon EC2 Auto Scaling ensures that your group never goes below this size\. You can specify the maximum number of instances in each Auto Scaling group, and Amazon EC2 Auto Scaling ensures that your group never goes above this size\. If you specify the desired capacity, either when you create the group or at any time thereafter, Amazon EC2 Auto Scaling ensures that your group has this many instances\. If you specify scaling policies, then Amazon EC2 Auto Scaling can launch or terminate instances as demand on your application increases or decreases\.

For example, the following Auto Scaling group has a minimum size of one instance, a desired capacity of two instances, and a maximum size of four instances\. The scaling policies that you define adjust the number of instances, within your minimum and maximum number of instances, based on the criteria that you specify\.

![\[An illustration of a basic Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-basic-diagram.png)

For more information about the benefits of Amazon EC2 Auto Scaling, see [Amazon EC2 Auto Scaling benefits](auto-scaling-benefits.md)\.

## Auto Scaling components<a name="as-component-intro"></a>

The following table describes the key components of Amazon EC2 Auto Scaling\.


****  

|  |  | 
| --- |--- |
|  ![\[A graphic representing an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/group-graphic.png)  |   Groups Your EC2 instances are organized into *groups* so that they can be treated as a logical unit for the purposes of scaling and management\. When you create a group, you can specify its minimum, maximum, and, desired number of EC2 instances\. For more information, see [Auto Scaling groups](auto-scaling-groups.md)\.   | 
|  ![\[A graphic representing a launch template or launch configuration.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/launch-configuration-graphic.png)  |   Configuration templates Your group uses a *launch template*, or a *launch configuration* \(not recommended, offers fewer features\), as a configuration template for its EC2 instances\. You can specify information such as the AMI ID, instance type, key pair, security groups, and block device mapping for your instances\. For more information, see [Launch templates](launch-templates.md) and [Launch configurations](launch-configurations.md)\.   | 
|  ![\[A graphic representing scaling options.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/scaling-plan-graphic.png)  |   Scaling options Amazon EC2 Auto Scaling provides several ways for you to scale your Auto Scaling groups\. For example, you can configure a group to scale based on the occurrence of specified conditions \(dynamic scaling\) or on a schedule\. For more information, see [Scaling options](scale-your-group.md#scaling-options)\.   | 

## Pricing for Amazon EC2 Auto Scaling<a name="as-pricing"></a>

There are no additional fees with Amazon EC2 Auto Scaling, so it's easy to try it out and see how it can benefit your AWS architecture\. You only pay for the AWS resources \(for example, EC2 instances, EBS volumes, and CloudWatch alarms\) that you use\.

## Get started<a name="what-is-auto-scaling-next-steps"></a>

To begin, complete the [Get started with Amazon EC2 Auto Scaling](get-started-with-ec2-auto-scaling.md) tutorial to create an Auto Scaling group and see how it responds when an instance in that group terminates\.

## Related services<a name="related-services"></a>

To automatically distribute incoming application traffic across multiple instances in your Auto Scaling group, use Elastic Load Balancing\. For more information, see [Use Elastic Load Balancing to distribute traffic across the instances in your Auto Scaling group](autoscaling-load-balancer.md)\.

To monitor your Auto Scaling groups and instance utilization data, use Amazon CloudWatch\. For more information, see [Monitor CloudWatch metrics for your Auto Scaling groups and instances](ec2-auto-scaling-cloudwatch-monitoring.md)\.

To configure auto scaling for scalable resources for Amazon Web Services beyond Amazon EC2, see the [Application Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/application/userguide/)\.

## Work with Auto Scaling groups<a name="auto-scaling-group-interfaces"></a>

You can create, access, and manage your Auto Scaling groups using any of the following interfaces:
+ **AWS Management Console** – Provides a web interface that you can use to access your Auto Scaling groups\. If you've signed up for an AWS account, you can access your Auto Scaling groups by signing into the AWS Management Console, using the search box on the navigation bar to search for **Auto Scaling groups**, and then choosing **Auto Scaling groups**\.
+ **AWS Command Line Interface \(AWS CLI\)** – Provides commands for a broad set of AWS services, and is supported on Windows, macOS, and Linux\. To get started, see [AWS Command Line Interface User Guide](https://docs.aws.amazon.com/cli/latest/userguide/)\. For more information, see [autoscaling](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/index.html) in the *AWS CLI Command Reference*\.
+ **AWS Tools for Windows PowerShell** – Provides commands for a broad set of AWS products for those who script in the PowerShell environment\. To get started, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\. For more information, see the [AWS Tools for PowerShell Cmdlet Reference](https://docs.aws.amazon.com/powershell/latest/reference/Index.html)\.
+ **AWS SDKs** – Provides language\-specific API operations and takes care of many of the connection details, such as calculating signatures, handling request retries, and handling errors\. For more information, see [AWS SDKs](http://aws.amazon.com/tools/#SDKs)\.
+ **Query API** – Provides low\-level API actions that you call using HTTPS requests\. Using the Query API is the most direct way to access AWS services\. However, it requires your application to handle low\-level details such as generating the hash to sign the request, and handling errors\. For more information, see the [Amazon EC2 Auto Scaling API Reference](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/)\.
+ **AWS CloudFormation** – Supports creating resources for Amazon EC2 Auto Scaling using a CloudFormation template\. For more information, including examples of JSON and YAML templates for Amazon EC2 Auto Scaling resources, see the [Amazon EC2 Auto Scaling resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_AutoScaling.html) in the *AWS CloudFormation User Guide*\.

To connect programmatically to an AWS service, you use an endpoint\. For information about endpoints for calls to Amazon EC2 Auto Scaling, see [Amazon EC2 Auto Scaling endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/as.html) in the *AWS General Reference*\. 