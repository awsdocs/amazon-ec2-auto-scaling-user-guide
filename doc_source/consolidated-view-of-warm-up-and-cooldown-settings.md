# Available warm\-up and cooldown settings<a name="consolidated-view-of-warm-up-and-cooldown-settings"></a>

To help optimize scaling performance, choose the appropriate warm\-up and cooldown settings for your Auto Scaling group\. 

We recommend using the `DefaultInstanceWarmup` setting, which unifies all the warm\-up and cooldown settings\. If needed, you can also use the `HealthCheckGracePeriod` setting\.

The other available settings are not intended to be used at the same time as the `DefaultInstanceWarmup` setting\. For example, `EstimatedInstanceWarmup` and `InstanceWarmup` aren't needed when `DefaultInstanceWarmup` is already defined, and `DefaultCooldown` and `Cooldown` are only needed when you use simple scaling policies\. \(As a best practice, we recommend target tracking and step scaling policies instead of simple scaling policies\.\) However, we will continue to support all of these settings so that you can choose the appropriate settings for your use case\. 

## `DefaultInstanceWarmup` \(Recommended\)<a name="consolidated-view-default-instance-warm-up"></a>

**API operation:** [CreateAutoScalingGroup](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateAutoScalingGroup.html), [UpdateAutoScalingGroup](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_UpdateAutoScalingGroup.html)

The amount of time, in seconds, until a newly launched instance can contribute to the Amazon CloudWatch metrics\. This delay lets an instance finish initializing before Amazon EC2 Auto Scaling aggregates instance metrics, resulting in more reliable usage data\. Set this value equal to the amount of time that it takes for resource consumption to become stable after an instance reaches the `InService` state\. For more information, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.

You can reduce the value of the instance warmup if you use a lifecycle hook for instance launch\. If your Auto Scaling group is behind a load balancer, you can add a lifecycle hook to the group to make sure that your instances are ready to serve traffic before they are registered to the load balancer at the end of the lifecycle hook\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

**Important**  
To manage your warm\-up settings at the group level, we recommend that you set the default instance warmup, *even if its value is set to 0 seconds*\. This also optimizes the performance of scaling policies that scale continuously, such as target tracking and step scaling policies\.   
If you need to remove a value that you previously set, include the property but specify `-1` for the value\. However, we strongly recommend keeping the default instance warmup enabled by specifying a minimum value of `0`\.

**Default**: None 

## `EstimatedInstanceWarmup`<a name="consolidated-view-estimated-instance-warm-up"></a>

*Not needed if `DefaultInstanceWarmup` is defined\.*

**API operation:** [PutScalingPolicy](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutScalingPolicy.html)

The estimated time, in seconds, until a newly launched instance can contribute to the CloudWatch metrics\. This warm\-up period applies to instances launched due to a specific target tracking or step scaling policy\. When a warm\-up period is specified here, it overrides the default instance warmup\. For more information, see [Target tracking scaling policies](as-scaling-target-tracking.md) and [Step and simple scaling policies](as-scaling-simple-step.md)\.

**Default**: If not defined, the value of this parameter defaults to the value for the default instance warmup defined for the group\. If default instance warmup is null, then it falls back to the value of default cooldown\.

## `InstanceWarmup`<a name="consolidated-view-instance-warm-up"></a>

*Not needed if `DefaultInstanceWarmup` is defined\.*

**API operation:** [StartInstanceRefresh](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_StartInstanceRefresh.html)

A warm\-up period, in seconds, that applies to a specific instance refresh operation\. Specifying a warm\-up period when you start an instance refresh overrides the default instance warmup, but only for the current instance refresh\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

**Default**: If not defined, the value of this parameter defaults to the value for the default instance warmup defined for the group\. If default instance warmup is null, then it falls back to the value of the health check grace period\.

## `DefaultCooldown`<a name="consolidated-view-default-cooldown"></a>

*Only needed if you use simple scaling policies\.*

**API operation:** [CreateAutoScalingGroup](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateAutoScalingGroup.html), [UpdateAutoScalingGroup](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_UpdateAutoScalingGroup.html)

The amount of time, in seconds, between one scaling activity ending and another one starting due to simple scaling policies\. For more information, see [Scaling cooldowns for Amazon EC2 Auto Scaling](ec2-auto-scaling-scaling-cooldowns.md)\.

**Default**: `300` seconds

## `Cooldown`<a name="consolidated-view-cooldown"></a>

*Only needed if you use simple scaling policies\.*

**API operation:** [PutScalingPolicy](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutScalingPolicy.html)

A cooldown period, in seconds, that applies to a specific simple scaling policy\. \(Simple scaling policies are no longer recommended\. As a best practice, we recommend target tracking and step scaling policies instead\.\) When a cooldown period is specified here, it overrides the default cooldown\. For more information, see [Scaling cooldowns for Amazon EC2 Auto Scaling](ec2-auto-scaling-scaling-cooldowns.md)\.

**Default**: None

## `HealthCheckGracePeriod`<a name="consolidated-view-health-check-grace-period"></a>

**API operation:** [CreateAutoScalingGroup](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateAutoScalingGroup.html), [UpdateAutoScalingGroup](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_UpdateAutoScalingGroup.html)

The amount of time, in seconds, that Amazon EC2 Auto Scaling waits before checking the health status of an EC2 instance that has come into service and marking it unhealthy due to a failed Elastic Load Balancing or custom health check\. This is useful if your instances do not immediately pass these health checks after they enter the `InService` state\. For more information, see [Health check grace period](ec2-auto-scaling-health-checks.md#health-check-grace-period)\.

You can set the value of the health check grace period to `0` if you use a lifecycle hook for instance launch\. If your Auto Scaling group is behind a load balancer, you can add a lifecycle hook to the group to make sure that your instances are ready to serve traffic before they are registered to the load balancer at the end of the lifecycle hook\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

**Default**: `300` seconds for an Auto Scaling group that you create in the AWS Management Console\. `0` seconds for an Auto Scaling group that you create using the AWS CLI or an SDK\.