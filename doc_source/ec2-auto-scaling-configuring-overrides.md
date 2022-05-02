# Configure overrides<a name="ec2-auto-scaling-configuring-overrides"></a>

Overrides are changes you make to the properties of your launch template\. Amazon EC2 Auto Scaling supports overrides to the instance type property\. That way, you can specify multiple instance types\. To do so, you must add the `Overrides` structure to your mixed instances policy\.

The `Overrides` structure allows you to define a set of parameters that Amazon EC2 Auto Scaling can use to launch instances, including:
+ `InstanceType` — The instance type\. For more information about the instance types that are available, see [Instance types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) in the *Amazon EC2 User Guide for Linux Instances*\. 
+ `LaunchTemplateSpecification` — \(Optional\) A launch template for an individual instance type\. This property is currently limited to the AWS CLI or an SDK, and is not available from the console\. 
+ `WeightedCapacity` — \(Optional\) The number of units that a provisioned instance of this type provides toward fulfilling the desired capacity of the Auto Scaling group\. If you specify a weight value for one instance type, you must specify a weight value for all of them\.

**Note**  
As an alternative to specifying a list of instance types, you can specify a set of instance attributes to use as criteria for selecting the instance types your Auto Scaling group uses\. This is known as *attribute\-based instance type selection*\. For more information, see [Using attribute\-based instance type selection](create-asg-instance-type-requirements.md)\.

**Topics**
+ [Specify a different launch template for an instance type](ec2-auto-scaling-mixed-instances-groups-launch-template-overrides.md)
+ [Configure instance weighting for Amazon EC2 Auto Scaling](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md)