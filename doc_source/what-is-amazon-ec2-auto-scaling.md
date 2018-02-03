# What Is Amazon EC2 Auto Scaling?<a name="what-is-amazon-ec2-auto-scaling"></a>

Amazon EC2 Auto Scaling helps you ensure that you have the correct number of Amazon EC2 instances available to handle the load for your application\. You create collections of EC2 instances, called *Auto Scaling groups*\. You can specify the minimum number of instances in each Auto Scaling group, and Auto Scaling ensures that your group never goes below this size\. You can specify the maximum number of instances in each Auto Scaling group, and Auto Scaling ensures that your group never goes above this size\. If you specify the desired capacity, either when you create the group or at any time thereafter, Auto Scaling ensures that your group has this many instances\. If you specify scaling policies, then Auto Scaling can launch or terminate instances as demand on your application increases or decreases\.

For example, the following Auto Scaling group has a minimum size of 1 instance, a desired capacity of 2 instances, and a maximum size of 4 instances\. The scaling policies that you define adjust the number of instances, within your minimum and maximum number of instances, based on the criteria that you specify\.

![\[An illustration of a basic Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-basic-diagram.png)

For more information about the benefits of Auto Scaling, see [Benefits of Auto Scaling](auto-scaling-benefits.md)\.

## Auto Scaling Components<a name="as-component-intro"></a>

The following table describes the key components of Auto Scaling\.


****  

|  |  | 
| --- |--- |
|  ![\[A graphic representing an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/group-graphic.png)  |   Groups Your EC2 instances are organized into *groups* so that they can be treated as a logical unit for the purposes of scaling and management\. When you create a group, you can specify its minimum, maximum, and, desired number of EC2 instances\. For more information, see [Auto Scaling Groups](AutoScalingGroup.md)\.   | 
|  ![\[A graphic representing a launch configuration.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/launch-configuration-graphic.png)  |   Launch configurations Your group uses a *launch configuration* as a template for its EC2 instances\. When you create a launch configuration, you can specify information such as the AMI ID, instance type, key pair, security groups, and block device mapping for your instances\. For more information, see [Launch Configurations](LaunchConfiguration.md)\.   | 
|  ![\[A graphic representing a launch configuration.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/scaling-plan-graphic.png)  |   Scaling plans A *scaling plan* tells Auto Scaling when and how to scale\. For example, you can base a scaling plan on the occurrence of specified conditions \(dynamic scaling\) or on a schedule\. For more information, see [Scaling Options](scaling_plan.md#scaling_typesof)\.   | 

## Getting Started<a name="what-is-auto-scaling-next-steps"></a>

If you're new to Auto Scaling, we recommend that you review [Auto Scaling Lifecycle](AutoScalingGroupLifecycle.md) before you begin\.

To begin, complete the [Getting Started with Amazon EC2 Auto Scaling](GettingStartedTutorial.md) tutorial to create an Auto Scaling group and see how it responds when an instance in that group terminates\. If you already have running EC2 instances, you can create an Auto Scaling group using an existing EC2 instance, and remove the instance from the group at any time\.

## Accessing Auto Scaling<a name="access-as"></a>

AWS provides a web\-based user interface, the AWS Management Console\. If you've signed up for an AWS account, you can access Auto Scaling by signing into the AWS Management Console\. To get started, choose **EC2** from the console home page, and then choose **Launch Configurations** from the navigation pane\.

If you prefer to use a command line interface, you have the following options:

**AWS Command Line Interface \(CLI\)**  
Provides commands for a broad set of AWS products, and is supported on Windows, Mac, and Linux\. To get started, see [AWS Command Line Interface User Guide](http://docs.aws.amazon.com/cli/latest/userguide/)\. For more information about the commands for Auto Scaling, see [autoscaling](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/index.html) in the *AWS Command Line Interface Reference*\.

**AWS Tools for Windows PowerShell**  
Provides commands for a broad set of AWS products for those who script in the PowerShell environment\. To get started, see the [AWS Tools for Windows PowerShell User Guide](http://docs.aws.amazon.com/powershell/latest/userguide/)\. For more information about the cmdlets for Auto Scaling, see the [AWS Tools for Windows PowerShell Reference](http://docs.aws.amazon.com/powershell/latest/reference/Index.html)\.

Auto Scaling provides a Query API\. These requests are HTTP or HTTPS requests that use the HTTP verbs GET or POST and a Query parameter named `Action`\. For more information about the API actions for Auto Scaling, see [Actions](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_Operations.html) in the *Amazon EC2 Auto Scaling API Reference*\.

If you prefer to build applications using language\-specific APIs instead of submitting a request over HTTP or HTTPS, AWS provides libraries, sample code, tutorials, and other resources for software developers\. These libraries provide basic functions that automate tasks such as cryptographically signing your requests, retrying requests, and handling error responses, making it is easier for you to get started\. For more information, see [AWS SDKs and Tools](http://aws.amazon.com/tools/)\.

For information about your credentials for accessing AWS, see [AWS Security Credentials](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html) in the *Amazon Web Services General Reference*\.

## Pricing for Auto Scaling<a name="as-pricing"></a>

There are no additional fees with Auto Scaling, so it's easy to try it out and see how it can benefit your AWS architecture\.

## PCI DSS Compliance<a name="pci-dss-compliance"></a>

Auto Scaling supports the processing, storage, and transmission of credit card data by a merchant or service provider, and has been validated as being compliant with Payment Card Industry \(PCI\) Data Security Standard \(DSS\)\. For more information about PCI DSS, including how to request a copy of the AWS PCI Compliance Package, see [PCI DSS Level 1](https://aws.amazon.com/compliance/pci-dss-level-1-faqs/)\. 

## Related Services<a name="related-services"></a>

To configure automatic scaling for all of the scalable resources for your application, use AWS Auto Scaling\. For more information, see the [AWS Auto Scaling User Guide](http://docs.aws.amazon.com/autoscaling/plans/userguide/)\.

To automatically distribute incoming application traffic across multiple instances in your Auto Scaling group, use Elastic Load Balancing\. For more information, see the [Elastic Load Balancing User Guide](http://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/)\.

To monitor basic statistics for your instances and Amazon EBS volumes, use Amazon CloudWatch\. For more information, see the [Amazon CloudWatch User Guide](http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.

To monitor the calls made to the Auto Scaling API for your account, including calls made by the AWS Management Console, command line tools, and other services, use AWS CloudTrail\. For more information, see the [AWS CloudTrail User Guide](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.