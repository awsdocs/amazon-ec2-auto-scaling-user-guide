# Scaling Cooldowns<a name="Cooldown"></a>

The cooldown period is a configurable setting for your Auto Scaling group that helps to ensure that it doesn't launch or terminate additional instances before the previous scaling activity takes effect\. After the Auto Scaling group dynamically scales using a simple scaling policy, it waits for the cooldown period to complete before resuming scaling activities\. When you manually scale your Auto Scaling group, the default is not to wait for the cooldown period, but you can override the default and honor the cooldown period\. If an instance becomes unhealthy, the Auto Scaling group does not wait for the cooldown period to complete before replacing the unhealthy instance\.

**Important**  
Cooldown periods are not supported for step scaling policies or scheduled scaling\.

Auto Scaling supports both default cooldown periods and scaling\-specific cooldown periods\.


+ [Example: Cooldowns](#cooldown-example)
+ [Default Cooldowns](#cooldown-default)
+ [Scaling\-Specific Cooldowns](#cooldowns-scaling-specific)
+ [Cooldowns and Multiple Instances](#cooldowns-multiple-instances)
+ [Cooldowns and Lifecycle Hooks](#cooldowns-lifecycle-hooks)
+ [Cooldowns and Spot Instances](#cooldowns-spot)

## Example: Cooldowns<a name="cooldown-example"></a>

Consider the following scenario: you have a web application running in AWS\. This web application consists three basic tiers: web, application, and database\. To make sure that the application always has the resources to meet traffic demands, you create two Auto Scaling groups: one for your web tier and one for your application tier\.

![\[A basic network architecture with a web tier and application
                        tier.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/cooldown-example-start-diagram.png)

To help ensure that the Auto Scaling group for the application tier has the appropriate number of EC2 instances, create a CloudWatch alarm to scale out whenever the **CPUUtilization** metric for the instances exceeds 90 percent\. When the alarm occurs, the Auto Scaling group launches and configures another instance\.

![\[An example of how a CloudWatch alarm works with a scaling policy\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/cooldowns-example-scaling-policy-diagram.png)

These instances use a configuration script to install and configure software before the instance is put into service\. As a result, it takes around two or three minutes from the time the instance launches until it comes in service\. The actual time depends on several factors, such as the size of the instance and whether there are startup scripts to complete\.

Now a spike in traffic occurs, causing the CloudWatch alarm to fire\. When it does, the Auto Scaling group launches an instance to help with the increase in demand\. However, there's a problem: the instance takes a couple of minutes to launch\. During that time, the CloudWatch alarm could continue to fire, causing the Auto Scaling group to launch another instance each time the alarm fires\.

However, with a cooldown period in place, the Auto Scaling group launches an instance and then suspends scaling activities due to simple scaling policies or manual scaling until the specified time elapses\. \(The default is 300 seconds\.\) This gives newly launched instances time to start handling application traffic\. After the cooldown period expires, any suspended scaling actions resume\. If the CloudWatch alarm fires again, the Auto Scaling group launches another instance, and the cooldown period takes effect again\. If, however, the additional instance was enough to bring the CPU utilization back down, then the group remains at its current size\.

## Default Cooldowns<a name="cooldown-default"></a>

The default cooldown period is applied when you create your Auto Scaling group\. Its default value is 300 seconds\. This cooldown period automatically applies to any dynamic scaling activities for simple scaling policies, and you can optionally request that it apply to your manual scaling activities\.

![\[A flowchart showing how a default cooldown affects scaling
                        actions.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/cooldowns-default-diagram.png)

You can configure the default cooldown period when you create the Auto Scaling group, using the AWS Management Console, the [create\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command \(AWS CLI\), or the [CreateAutoScalingGroup](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateAutoScalingGroup.html) API operation\.

You can change the default cooldown period at any time, using the AWS Management Console, the [update\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command \(AWS CLI\), or the [UpdateAutoScalingGroup](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_UpdateAutoScalingGroup.html) API operation\.

## Scaling\-Specific Cooldowns<a name="cooldowns-scaling-specific"></a>

In addition to specifying the default cooldown period for your Auto Scaling group, you can create cooldowns that apply to a specific simple scaling policy or manual scaling\. A scaling\-specific cooldown period overrides the default cooldown period\.

One common use for scaling\-specific cooldowns is with a scale\-in policy—a policy that terminates instances based on a specific criteria or metric\. Because this policy terminates instances, Amazon EC2 Auto Scaling needs less time to determine whether to terminate additional instances\. The default cooldown period of 300 seconds is too long—you can reduce costs by applying a scaling\-specific cooldown period of 180 seconds to the scale\-in policy\.

You can create a scaling\-specific cooldown period using the AWS Management Console, the [put\-scaling\-policy](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command \(AWS CLI\), or the [PutScalingPolicy](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutScalingPolicy.html) API operation\.

## Cooldowns and Multiple Instances<a name="cooldowns-multiple-instances"></a>

The preceding sections have provided examples that show how cooldown periods affect Auto Scaling groups when a single instance launches or terminates\. However, it is not uncommon for Auto Scaling groups to launch more than one instance at a time\. For example, you might choose to have the Auto Scaling group launch three instances when a specific metric threshold is met\.

With multiple instances, the cooldown period \(either the default cooldown or the scaling\-specific cooldown\) takes effect starting when the last instance launches\.

## Cooldowns and Lifecycle Hooks<a name="cooldowns-lifecycle-hooks"></a>

You can add lifecycle hooks to your Auto Scaling groups\. These hooks enable you to control how instances launch and terminate within an Auto Scaling group; you can perform actions on the instance before it is put into service or before it is terminated\.

Lifecycle hooks can affect the impact of any cooldown periods configured for the Auto Scaling group, manual scaling, or a simple scaling policy\. The cooldown period does not begin until after the instance moves out of the wait state\.

## Cooldowns and Spot Instances<a name="cooldowns-spot"></a>

You can create Auto Scaling groups to use Spot Instances instead of On\-Demand or Reserved Instances\. The cooldown period begins when the bid for any Spot Instance is successful\.