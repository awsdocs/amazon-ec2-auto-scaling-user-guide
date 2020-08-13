# What is Amazon EC2 Auto Scaling?<a name="what-is-amazon-ec2-auto-scaling"></a>

Amazon EC2 Auto Scaling helps you ensure that you have the correct number of Amazon EC2 instances available to handle the load for your application\. You create collections of EC2 instances, called *Auto Scaling groups*\. You can specify the minimum number of instances in each Auto Scaling group, and Amazon EC2 Auto Scaling ensures that your group never goes below this size\. You can specify the maximum number of instances in each Auto Scaling group, and Amazon EC2 Auto Scaling ensures that your group never goes above this size\. If you specify the desired capacity, either when you create the group or at any time thereafter, Amazon EC2 Auto Scaling ensures that your group has this many instances\. If you specify scaling policies, then Amazon EC2 Auto Scaling can launch or terminate instances as demand on your application increases or decreases\.

For example, the following Auto Scaling group has a minimum size of one instance, a desired capacity of two instances, and a maximum size of four instances\. The scaling policies that you define adjust the number of instances, within your minimum and maximum number of instances, based on the criteria that you specify\.

![\[An illustration of a basic Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-basic-diagram.png)

For more information about the benefits of Amazon EC2 Auto Scaling, see [Benefits of Amazon EC2 Auto Scaling](auto-scaling-benefits.md)\.

## Auto Scaling components<a name="as-component-intro"></a>

The following table describes the key components of Amazon EC2 Auto Scaling\.


****  

|  |  | 
| --- |--- |
|  ![\[A graphic representing an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/group-graphic.png)  |   Groups Your EC2 instances are organized into *groups* so that they can be treated as a logical unit for the purposes of scaling and management\. When you create a group, you can specify its minimum, maximum, and, desired number of EC2 instances\. For more information, see [Auto Scaling groups](AutoScalingGroup.md)\.   | 
|  ![\[A graphic representing a launch configuration.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/launch-configuration-graphic.png)  |   Configuration templates Your group uses a *launch template* or a *launch configuration* as a configuration template for its EC2 instances\. You can specify information such as the AMI ID, instance type, key pair, security groups, and block device mapping for your instances\. For more information, see [Launch templates](LaunchTemplates.md) and [Launch configurations](LaunchConfiguration.md)\.   | 
|  ![\[A graphic representing scaling options.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/scaling-plan-graphic.png)  |   Scaling options Amazon EC2 Auto Scaling provides several ways for you to scale your Auto Scaling groups\. For example, you can configure a group to scale based on the occurrence of specified conditions \(dynamic scaling\) or on a schedule\. For more information, see [Scaling options](scaling_plan.md#scaling_typesof)\.   | 

## Getting started<a name="what-is-auto-scaling-next-steps"></a>

If you're new to Amazon EC2 Auto Scaling, we recommend that you review [Auto Scaling instance lifecycle](AutoScalingGroupLifecycle.md) before you begin\.

To begin, complete the [Getting started with Amazon EC2 Auto Scaling](GettingStartedTutorial.md) tutorial to create an Auto Scaling group and see how it responds when an instance in that group terminates\. If you already have running EC2 instances, you can create an Auto Scaling group using an existing EC2 instance, and remove the instance from the group at any time\.

## Accessing Amazon EC2 Auto Scaling<a name="access-as"></a>

If you've signed up for an AWS account, you can access Amazon EC2 Auto Scaling by signing into the AWS Management Console, choosing **EC2** from the console home page, and then choosing **Auto Scaling Groups** from the navigation pane\.

You can also access Amazon EC2 Auto Scaling using the [Amazon EC2 Auto Scaling API](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/)\. Amazon EC2 Auto Scaling provides a Query API\. These requests are HTTP or HTTPS requests that use the HTTP verbs GET or POST and a Query parameter named `Action`\. For more information about the API actions for Amazon EC2 Auto Scaling, see [Actions](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_Operations.html) in the *Amazon EC2 Auto Scaling API Reference*\.

If you prefer to build applications using language\-specific APIs instead of submitting a request over HTTP or HTTPS, AWS provides libraries, sample code, tutorials, and other resources for software developers\. These libraries provide basic functions that automate tasks such as cryptographically signing your requests, retrying requests, and handling error responses, making it is easier for you to get started\. For more information, see [AWS SDKs and tools](https://aws.amazon.com/tools/)\.

If you prefer to use a command line interface, you have the following options:

**AWS Command Line Interface \(CLI\)**  
Provides commands for a broad set of AWS products, and is supported on Windows, macOS, and Linux\. To get started, see [AWS Command Line Interface User Guide](https://docs.aws.amazon.com/cli/latest/userguide/)\. For more information, see [autoscaling](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/index.html) in the *AWS CLI Command Reference*\.

**AWS Tools for Windows PowerShell**  
Provides commands for a broad set of AWS products for those who script in the PowerShell environment\. To get started, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\. For more information, see the [AWS Tools for PowerShell Cmdlet Reference](https://docs.aws.amazon.com/powershell/latest/reference/Index.html)\.

For information about your credentials for accessing AWS, see [AWS security credentials](https://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html) in the *Amazon Web Services General Reference*\. For information about regions and endpoints for calls to Amazon EC2 Auto Scaling, see the [Regions and endpoints](https://docs.aws.amazon.com/general/latest/gr/as.html) table in the *AWS General Reference*\. 

## Pricing for Amazon EC2 Auto Scaling<a name="as-pricing"></a>

There are no additional fees with Amazon EC2 Auto Scaling, so it's easy to try it out and see how it can benefit your AWS architecture\.

## PCI DSS compliance<a name="pci-dss-compliance"></a>

Auto Scaling supports the processing, storage, and transmission of credit card data by a merchant or service provider, and has been validated as being compliant with Payment Card Industry \(PCI\) Data Security Standard \(DSS\)\. For more information about PCI DSS, including how to request a copy of the AWS PCI Compliance Package, see [PCI DSS Level 1](https://aws.amazon.com/compliance/pci-dss-level-1-faqs/)\. 

## Related services<a name="related-services"></a>

To configure automatic scaling for all of the scalable AWS resources for your application, use AWS Auto Scaling\. With AWS Auto Scaling, you can also simplify the process of defining dynamic scaling policies for your Auto Scaling groups and use predictive scaling to scale your Amazon EC2 capacity in advance of predicted traffic changes\. For more information, see the [AWS Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/plans/userguide/)\. 

To automatically distribute incoming application traffic across multiple instances in your Auto Scaling group, use Elastic Load Balancing\. For more information, see the [Elastic Load Balancing User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/)\.

To monitor basic statistics for your instances and Amazon EBS volumes, use Amazon CloudWatch\. For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.

To monitor the calls made to the Amazon EC2 Auto Scaling API for your account, use AWS CloudTrail\. The data logged includes calls made by the AWS Management Console, command line tools, and other services\. For more information, see the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.