# Understand the default values for an instance refresh<a name="understand-instance-refresh-default-values"></a>

Before you start an instance refresh, you can customize various preferences that affect the instance refresh\. Some preference defaults are different depending on whether you use the console or the command line \(AWS CLI or AWS SDK\)\.

The following table lists the default values for the instance refresh settings\.


| Setting | AWS CLI or AWS SDK | Amazon EC2 Auto Scaling console | 
| --- | --- | --- | 
| Auto rollback | Disabled \(false\) | Disabled | 
| Checkpoints | Disabled \(false\) | Disabled | 
| Checkpoint delay | 1 hour \(3600 seconds\) | 1 hour | 
| Instance warmup | The [default instance warmup](ec2-auto-scaling-default-instance-warmup.md), if defined, or the [health check grace period](health-check-grace-period.md) otherwise\. | The [default instance warmup](ec2-auto-scaling-default-instance-warmup.md), if defined, or the [health check grace period](health-check-grace-period.md) otherwise\. | 
| Minimum healthy percentage | 90 | 90 | 
| Scale\-in protected instances | Wait | Ignore | 
| Skip matching | Disabled \(false\) | Enabled | 
| Standby instances | Wait | Ignore | 

A description of each setting follows:

**Auto rollback \(`AutoRollback`\)**  
Choose whether to roll back the Auto Scaling group to its previous configuration if the instance refresh fails\. For more information, see [Undo changes with a rollback](instance-refresh-rollback.md)\. 

**Checkpoints \(`CheckpointPercentages`\)**  
Choose whether to replace instances in phases\. This way, you can perform verifications on your instances as you go\. For more information, see [Add checkpoints to an instance refresh](asg-adding-checkpoints-instance-refresh.md)\. 

**Checkpoint delay \(`CheckpointDelay`\)**  
The amount of time, in seconds, to wait after a checkpoint before continuing\. For more information, see [Add checkpoints to an instance refresh](asg-adding-checkpoints-instance-refresh.md)\. 

**Instance warmup \(`InstanceWarmup`\)**  
A time period, in seconds, during which Amazon EC2 Auto Scaling waits for an instance to be ready to receive traffic before moving on to replacing the next instance\. If you have already correctly defined a default instance warmup for the Auto Scaling group, then you don't need to change the instance warmup \(unless you want to override the default\)\. For more information, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.

**Minimum healthy percentage \(`MinHealthyPercentage`\)**  
The percentage of the desired capacity of the Auto Scaling group that must pass the group's health checks before the refresh can continue\. For more information about these health checks, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\.

**Scale\-in protected instances \(`ScaleInProtectedInstances`\)**  
Choose what Amazon EC2 Auto Scaling does if instances that are protected from scale in are found\. For more information about these instances, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.  
Amazon EC2 Auto Scaling provides the following options:  
+ **Replace** \(`Refresh`\) — Replaces instances that are protected from scale in\.
+ **Ignore** \(`Ignore`\) — Ignores instances that are protected from scale in and continues to replace instances that are not protected\.
+ **Wait** \(`Wait`\) — Waits one hour for you to remove scale\-in protection\. If you don't do so, the instance refresh fails\.

**Skip matching \(`SkipMatching`\)**  
Choose whether Amazon EC2 Auto Scaling skips replacing instances that match the desired configuration\. If no desired configuration is specified, then it skips replacing instances that have the same launch template and instance types that the Auto Scaling group was using before the instance refresh started\. For more information, see [Use an instance refresh with skip matching](asg-instance-refresh-skip-matching.md)\. 

**Standby instances \(`StandbyInstances`\)**  
Choose what Amazon EC2 Auto Scaling does if instances are found in `Standby` state\. For more information about these instances, see [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md)\.  
Amazon EC2 Auto Scaling provides the following options:  
+ **Terminate** \(`Terminate`\) — Terminates instances that are in `Standby`\.
+ **Ignore** \(`Ignore`\) — Ignores instances that are in `Standby` and continues to replace instances that are in the `InService` state\.
+ **Wait** \(`Wait`\) — Waits one hour for you to return the instances to service\. If you don't do so, the instance refresh fails\.