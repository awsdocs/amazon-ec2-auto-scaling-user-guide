# Scaling Cooldowns for Amazon EC2 Auto Scaling<a name="Cooldown"></a>

The cooldown period helps you prevent your Auto Scaling group from launching or terminating additional instances before the effects of previous activities are visible\. You can configure the length of time based on your instance start\-up time or other application needs\.

When you use a simple scaling policy for dynamic scaling, after the Auto Scaling group scales using a simple scaling policy, it waits for the cooldown period to complete before any further scaling activities due to simple scaling policies can start\. An adequate cooldown period helps to prevent the initiation of a scaling activity based on stale metrics\. By default, all simple scaling policies use the default cooldown period associated with your Auto Scaling group, but you can configure a different cooldown period for certain policies, as described in the following sections\. 

Note that Amazon EC2 Auto Scaling honors cooldown periods when using simple scaling policies, but not when using other scaling policies or scheduled scaling\. For more information about simple scaling, see [Step and Simple Scaling Policies](as-scaling-simple-step.md)\. 

**Important**  
In most cases, a target tracking scaling policy or a step scaling policy is more optimal for scaling performance than waiting for a fixed period of time to pass after there is a scaling activity\. For a scaling policy that changes the size of your Auto Scaling group proportionally as the value of the scaling metric decreases or increases, we recommend target tracking over either simple scaling or step scaling\.

When you manually scale your Auto Scaling group using the AWS CLI or AWS SDKs, the default is not to wait for the cooldown period to complete, but you can override this behavior and honor the cooldown period\. If an instance becomes unhealthy, the Auto Scaling group does not wait for the cooldown period to complete before replacing the unhealthy instance\. For more information about manual scaling, see [Manual Scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.

**Topics**
+ [Default Cooldown Period](#cooldown-default)
+ [Scaling\-Specific Cooldown Period](#cooldowns-scaling-specific)
+ [Example Simple Scaling Cooldown Scenario](#cooldown-example)
+ [Cooldowns and Multiple Instances](#cooldowns-multiple-instances)
+ [Cooldowns and Lifecycle Hooks](#cooldowns-lifecycle-hooks)

## Default Cooldown Period<a name="cooldown-default"></a>

A default cooldown period automatically applies to any scaling activities for simple scaling policies, and you can optionally request to have it apply to your manual scaling activities\. 

![\[A flowchart showing how a default cooldown affects scaling actions.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/cooldowns-default-diagram.png)

When you use the AWS Management Console to update an Auto Scaling group, or when you use the AWS CLI or an AWS SDK to create or update an Auto Scaling group, you can set the optional default cooldown parameter\. If a value for the default cooldown period is not provided, its default value is 300 seconds\. 

**To modify a default cooldown period \(console\)**  
Create the Auto Scaling group in the usual way\. After creating the Auto Scaling group, edit the group to specify the default cooldown period\. 

**To modify a default cooldown period \(AWS CLI\)**  
Use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html)
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html)

## Scaling\-Specific Cooldown Period<a name="cooldowns-scaling-specific"></a>

In addition to specifying the default cooldown period for your Auto Scaling group, you can create cooldowns that apply to a specific simple scaling policy and manual scaling\. A scaling\-specific cooldown period overrides the default cooldown period\.

One common use for a scaling\-specific cooldown period is with a scale\-in policy\. Because this policy terminates instances, Amazon EC2 Auto Scaling needs less time to determine whether to terminate additional instances\. Terminating instances should be a much quicker operation than launching instances\. The default cooldown period of 300 seconds is therefore too long\. In this case, a scaling\-specific cooldown period with a lower value of 180 seconds for your scale\-in policy can help you reduce costs by allowing the group to scale in faster\. 

To specify a scaling\-specific cooldown period, use the optional cooldown parameter when you create or update a simple scaling policy\. For more information, see [Step and Simple Scaling Policies](as-scaling-simple-step.md)\. 

## Example Simple Scaling Cooldown Scenario<a name="cooldown-example"></a>

Consider the following scenario: you have a web application running in AWS\. This web application consists of three basic tiers: web, application, and database\. To make sure that the application always has the resources to meet traffic demands, you create two Auto Scaling groups: one for your web tier and one for your application tier\.

![\[A basic network architecture with a web tier and application tier.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/cooldown-example-start-diagram.png)

To help ensure that the group for the application tier has the appropriate number of EC2 instances, you create a simple scaling policy to scale out whenever the value for the CloudWatch metric associated with the scaling policy exceeds a specified threshold for consecutive specified periods\. When the CloudWatch alarm triggers the scaling policy, the Auto Scaling group launches and configures another instance\.

These instances use a configuration script to install and configure software before the instance is put into service\. As a result, it takes around two or three minutes from the time the instance launches until it's fully in service\. The actual time depends on several factors, such as the size of the instance and whether there are startup scripts to complete\.

Now a spike in traffic occurs, causing the CloudWatch alarm to fire\. When it does, the Auto Scaling group launches an instance to help with the increase in demand\. However, there's a problem: the instance takes a couple of minutes to launch\. During that time, the CloudWatch alarm could continue to fire every minute for any standard resolution alarm, causing the Auto Scaling group to launch another instance each time the alarm fires\.

![\[An example of how a CloudWatch alarm works with a scaling policy\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/cooldowns-example-scaling-policy-diagram.png)

However, with a cooldown period in place, the Auto Scaling group launches an instance and then blocks scaling activities due to simple scaling policies until the specified time elapses\. \(The default is 300 seconds\.\) This gives newly launched instances time to start handling application traffic\. After the cooldown period expires, any scaling activities that are triggered after the cooldown period can resume\. If the CloudWatch alarm fires again, the Auto Scaling group launches another instance, and the cooldown period takes effect again\. However, if the additional instance was enough to bring the metric value back down, the group remains at its current size\.

## Cooldowns and Multiple Instances<a name="cooldowns-multiple-instances"></a>

The preceding section provides examples that show how cooldown periods affect Auto Scaling groups when a single instance launches or terminates\. However, it is common for Auto Scaling groups to launch more than one instance at a time\. For example, you might choose to have the Auto Scaling group launch three instances when a specific metric threshold is met\.

With multiple instances, the cooldown period \(either the default cooldown or the scaling\-specific cooldown\) takes effect starting when the last instance finishes launching or terminating\.

## Cooldowns and Lifecycle Hooks<a name="cooldowns-lifecycle-hooks"></a>

You have the option to add lifecycle hooks to your Auto Scaling groups\. These hooks enable you to control how instances launch and terminate within an Auto Scaling group so that you can perform custom actions on an instance before it is put into service or before it is terminated\. For more information, see [Amazon EC2 Auto Scaling Lifecycle Hooks](lifecycle-hooks.md)\.

Lifecycle hooks can affect the impact of any cooldown periods configured for the Auto Scaling group\. The cooldown period does not begin until after the instance moves out of the wait state \(after the lifecycle hook execution is complete\)\.

Consider an Auto Scaling group that has a lifecycle hook that supports a custom action at instance launch\. When the application experiences an increase in demand due to a simple scaling policy, the group launches an instance to add capacity\. Because there is a lifecycle hook, the instance is put into the `Pending:Wait` state, which means that it is not available to handle traffic yet\. When the instance enters the wait state, scaling activities due to simple scaling policies are paused\. When the instance enters the `InService` state, the cooldown period starts\. When the cooldown period expires, any scaling activities that are triggered after the cooldown period can resume\.