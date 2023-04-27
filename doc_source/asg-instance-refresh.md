# Replace Auto Scaling instances based on an instance refresh<a name="asg-instance-refresh"></a>

You can use an instance refresh to update the instances in your Auto Scaling group instead of manually replacing instances a few at a time\. This can be useful when a configuration change requires you to replace instances, and you have a large number of instances in your Auto Scaling group\. 

An instance refresh can be helpful when you have a new Amazon Machine Image \(AMI\) or a new user data script\. To use an instance refresh, first create a new launch template that specifies the new AMI or user data script\. Then, start an instance refresh to begin updating the instances in the group immediately\. 

An instance refresh can also be helpful when you are migrating your Auto Scaling groups from launch configurations to launch templates\. First, copy your launch configurations to new launch templates\. Then, start an instance refresh that specifies the launch template as part of the desired configuration to begin updating the instances in the group immediately\. For more information about migrating to launch templates, see [Migrate to launch templates](launch-templates.md#migrate-to-launch-templates)\.

**Topics**
+ [How it works](#instance-refresh-how-it-works)
+ [Core concepts and terms](#instance-refresh-core-concepts)
+ [Instance type compatibility](#instance-type-compatibility)
+ [Limitations](#instance-refresh-limitations)
+ [Start an instance refresh](start-instance-refresh.md)
+ [Understand the default values](understand-instance-refresh-default-values.md)
+ [Check instance refresh status](check-status-instance-refresh.md)
+ [Cancel an instance refresh](cancel-instance-refresh.md)
+ [Undo changes with a rollback](instance-refresh-rollback.md)
+ [Use skip matching](asg-instance-refresh-skip-matching.md)
+ [Add checkpoints](asg-adding-checkpoints-instance-refresh.md)

## How it works<a name="instance-refresh-how-it-works"></a>

The following example steps show how you might use an instance refresh to update your Auto Scaling group with a new launch template:
+ You create a new launch template for your Auto Scaling group\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\.
+ You configure the minimum healthy percentage, instance warmup, and checkpoints, specify your desired configuration that includes your launch template, and start an instance refresh\. The desired configuration can optionally specify whether a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md) is to be applied\.
+ Amazon EC2 Auto Scaling starts performing a rolling replacement of the instances\. It takes a set of instances out of service, terminates them, and launches a set of instances with the new desired configuration\. Then, it waits until the instances pass your health checks and complete warmup before it moves on to replacing other instances\. 
+ After a certain percentage of the group is replaced, a checkpoint is reached\. Whenever there is a checkpoint, Amazon EC2 Auto Scaling temporarily stops replacing instances and sends a notification\. Then, it waits for the amount of time you specified before continuing\. After you receive the notification, you can verify that your new instances are working as expected\.
+ After the instance refresh succeeds, the Auto Scaling group settings are automatically updated with the launch template that you specified at the start of the operation\. 

## Core concepts and terms<a name="instance-refresh-core-concepts"></a>

Before you get started, familiarize yourself with the following instance refresh core concepts and terms:

**Minimum healthy percentage**  
As part of starting an instance refresh, you specify the minimum healthy percentage to maintain at all times\. This is the amount of capacity in an Auto Scaling group that must pass your [health checks](ec2-auto-scaling-health-checks.md) during an instance refresh so that the refresh can continue\. For example, if the minimum healthy percentage is 90 percent, then 10 percent is the percentage of capacity that will be terminated and replaced\. If the new instances don't pass their health checks, Amazon EC2 Auto Scaling terminates and replaces them\. If it cannot launch any healthy instances, the instance refresh will eventually fail, leaving the other 90 percent of the group untouched\. If the new instances stay healthy and have finished their warm\-up period, Amazon EC2 Auto Scaling can continue to replace other instances\.  
An instance refresh can replace instances one at a time, several at a time, or all at once\. To replace one instance at a time, set a minimum healthy percentage of 100 percent\. To replace all at once, set a minimum healthy percentage of 0 percent\.  
Amazon EC2 Auto Scaling determines whether an instance is healthy based on the status of the health checks that your Auto Scaling group uses\. For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\. To make sure that Amazon EC2 Auto Scaling can detect instance health issues as soon as possible, don't set the group's health check grace period too high\. For more information, see [Set the health check grace period for an Auto Scaling group](health-check-grace-period.md)\.

**Instance warmup**  
Use the default instance warmup to define the warm\-up period for your Auto Scaling group\. Only specify the instance warmup for an instance refresh if you must override the default instance warmup\. The instance warmup works the same way as the default instance warmup so the same scaling considerations apply to the instance warmup\. For more information, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.
The *instance warmup* is the time period from when a new instance's state changes to `InService` to when it can receive traffic\. During an instance refresh, if the instances pass their health checks, Amazon EC2 Auto Scaling does not immediately move on to replacing the next instance after determining that a newly launched instance is healthy\. It waits for the warm\-up period before it moves on to replacing the next instance\. This can be helpful when your application takes time to initialize itself before it starts to serve traffic\.  
You can reduce the value of the warm\-up period if you use a lifecycle hook to prepare new instances for use\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

**Desired configuration**  
The *desired configuration* is the new configuration that you want Amazon EC2 Auto Scaling to deploy across your Auto Scaling group\. For example, you can specify a new launch template and new instance types for your instances\. During an instance refresh, Amazon EC2 Auto Scaling updates the Auto Scaling group to the desired configuration\. If a scale\-out event occurs during an instance refresh, Amazon EC2 Auto Scaling launches new instances with the desired configuration instead of the group's current settings\. After the instance refresh succeeds, Amazon EC2 Auto Scaling updates the Auto Scaling group settings to reflect the new desired configuration that you specified as part of the instance refresh\. 

**Skip matching**  
Skip matching tells Amazon EC2 Auto Scaling to ignore instances that already have your latest updates\. This way, you don't replace more instances than you need to\. This is helpful when you want to make sure your Auto Scaling group uses a particular version of your launch template and only replaces those instances that use a different version\.

**Checkpoints**  
A *checkpoint* is a point in time where the instance refresh pauses for a specified amount of time\. An instance refresh can contain multiple checkpoints\. Amazon EC2 Auto Scaling emits events for each checkpoint\. Therefore, you can add an EventBridge rule to send the events to a target, such as Amazon SNS, to be notified when a checkpoint is reached\. After a checkpoint is reached, you have the opportunity to verify your deployment\. If any problems are identified, you can cancel the instance refresh or roll it back\. The ability to deploy updates in phases is a key benefit of checkpoints\. If you don't use checkpoints, rolling replacements are performed continuously\.

## Instance type compatibility<a name="instance-type-compatibility"></a>

Before you change your instance type, it's a good idea to verify that it works with your launch template\. This confirms compatibility with the AMI that you specified\. For example, let's say you launched your original instances from a paravirtual \(PV\) AMI, but you want to change to a current generation instance type that is only supported by a hardware virtual machine \(HVM\) AMI\. In this case, you must use an HVM AMI in your launch template\. 

To confirm compatibility of the instance type without launching instances, use the [run\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html) command with the `--dry-run` option, as shown in the following example\.

```
aws ec2 run-instances --launch-template LaunchTemplateName=my-template,Version='1' --dry-run
```

For information about how compatibility is determined, see [Compatibility for changing the instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/resize-limitations.html) in the *Amazon EC2 User Guide for Linux Instances*\.

## Limitations<a name="instance-refresh-limitations"></a>
+ **Instances terminated before launch**: When there is only one instance in the Auto Scaling group, starting an instance refresh can result in an outage\. This is because Amazon EC2 Auto Scaling terminates an instance and then launches a new instance\.
+ **Total duration**: The maximum amount of time that an instance refresh can continue to actively replace instances is 14 days\. 
+ **Difference in behavior specific to weighted groups**: If a mixed instances group is configured with an instance weight that is larger than or equal to the group's desired capacity, Amazon EC2 Auto Scaling might replace all `InService` instances at once\. To avoid this situation, follow the recommendation in the [Configure instance weighting for Amazon EC2 Auto Scaling](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md) topic\. Specify a desired capacity that is larger than your largest weight when you use weights with your Auto Scaling group\.
+ **One\-hour timeout**: When an instance refresh is unable to continue making replacements because it is waiting to replace instances on standby or protected from scale in, or the new instances do not pass their health checks, Amazon EC2 Auto Scaling keeps retrying for an hour\. It also provides a status message to help you resolve the issue\. If the problem persists after an hour, the operation fails\. The intention is to give it time to recover if there is a temporary issue\. 