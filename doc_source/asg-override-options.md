# Configuring overrides<a name="asg-override-options"></a>

Depending on your requirements, you must override some attributes in the current launch template when using multiple instance types\. For example, some instance types might require a launch template with a different AMI\. To do so, you must configure the `Overrides` property\.

The `Overrides` property allows you to define a set of properties that Amazon EC2 Auto Scaling can use to launch instances, including:
+ `InstanceType` — The instance type\. For more information about the instance types that are available, see [Instance types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) in the *Amazon EC2 User Guide for Linux Instances*\. 
+ `LaunchTemplateSpecification` — \(Optional\) A launch template for an individual instance type\. This property is currently limited to the AWS CLI or an SDK, and is not available from the console\. 
+ `WeightedCapacity` — \(Optional\) The number of units that a provisioned instance of this type provides toward fulfilling the desired capacity of the Auto Scaling group\.

**Topics**
+ [Specifying a different launch template for an instance type](asg-launch-template-overrides.md)
+ [Configuring instance weighting for Amazon EC2 Auto Scaling](asg-instance-weighting.md)