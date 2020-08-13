# Auto Scaling instance lifecycle<a name="AutoScalingGroupLifecycle"></a>

The EC2 instances in an Auto Scaling group have a path, or lifecycle, that differs from that of other EC2 instances\. The lifecycle starts when the Auto Scaling group launches an instance and puts it into service\. The lifecycle ends when you terminate the instance, or the Auto Scaling group takes the instance out of service and terminates it\.

**Note**  
You are billed for instances as soon as they are launched, including the time that they are not yet in service\.

The following illustration shows the transitions between instance states in the Amazon EC2 Auto Scaling lifecycle\.

![\[The lifecycle of instances within an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/auto_scaling_lifecycle.png)

## Scale out<a name="as-lifecycle-scale-out"></a>

The following scale\-out events direct the Auto Scaling group to launch EC2 instances and attach them to the group:
+ You manually increase the size of the group\. For more information, see [Manual scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.
+ You create a scaling policy to automatically increase the size of the group based on a specified increase in demand\. For more information, see [Dynamic scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)\.
+ You set up scaling by schedule to increase the size of the group at a specific time\. For more information, see [Scheduled scaling for Amazon EC2 Auto Scaling](schedule_time.md)\.

When a scale\-out event occurs, the Auto Scaling group launches the required number of EC2 instances, using its assigned launch configuration\. These instances start in the `Pending` state\. If you add a lifecycle hook to your Auto Scaling group, you can perform a custom action here\. For more information, see [Lifecycle hooks](#as-lifecycle-hooks)\.

When each instance is fully configured and passes the Amazon EC2 health checks, it is attached to the Auto Scaling group and it enters the `InService` state\. The instance is counted against the desired capacity of the Auto Scaling group\.

## Instances in service<a name="as-lifecycle-inservice"></a>

Instances remain in the `InService` state until one of the following occurs:
+ A scale\-in event occurs, and Amazon EC2 Auto Scaling chooses to terminate this instance in order to reduce the size of the Auto Scaling group\. For more information, see [Controlling which Auto Scaling instances terminate during scale in](as-instance-termination.md)\.
+ You put the instance into a `Standby` state\. For more information, see [Enter and exit standby](#as-lifecycle-standby)\.
+ You detach the instance from the Auto Scaling group\. For more information, see [Detach an instance](#as-lifecycle-detach)\.
+ The instance fails a required number of health checks, so it is removed from the Auto Scaling group, terminated, and replaced\. For more information, see [Health checks for Auto Scaling instances](healthcheck.md)\.

## Scale in<a name="as-lifecycle-scale-in"></a>

The following scale\-in events direct the Auto Scaling group to detach EC2 instances from the group and terminate them:
+ You manually decrease the size of the group\. For more information, see [Manual scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.
+ You create a scaling policy to automatically decrease the size of the group based on a specified decrease in demand\. For more information, see [Dynamic scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)\.
+ You set up scaling by schedule to decrease the size of the group at a specific time\. For more information, see [Scheduled scaling for Amazon EC2 Auto Scaling](schedule_time.md)\.

It is important that you create a corresponding scale\-in event for each scale\-out event that you create\. This helps ensure that the resources assigned to your application match the demand for those resources as closely as possible\.

When a scale\-in event occurs, the Auto Scaling group detaches one or more instances\. The Auto Scaling group uses its termination policy to determine which instances to terminate\. Instances that are in the process of detaching from the Auto Scaling group and shutting down enter the `Terminating` state, and can't be put back into service\. If you add a lifecycle hook to your Auto Scaling group, you can perform a custom action here\. Finally, the instances are completely terminated and enter the `Terminated` state\.

## Attach an instance<a name="as-lifecycle-attach"></a>

You can attach a running EC2 instance that meets certain criteria to your Auto Scaling group\. After the instance is attached, it is managed as part of the Auto Scaling group\.

For more information, see [Attach EC2 instances to your Auto Scaling group](attach-instance-asg.md)\.

## Detach an instance<a name="as-lifecycle-detach"></a>

You can detach an instance from your Auto Scaling group\. After the instance is detached, you can manage it separately from the Auto Scaling group or attach it to a different Auto Scaling group\.

For more information, see [Detach EC2 instances from your Auto Scaling group](detach-instance-asg.md)\.

## Lifecycle hooks<a name="as-lifecycle-hooks"></a>

You can add a lifecycle hook to your Auto Scaling group so that you can perform custom actions when instances launch or terminate\.

When Amazon EC2 Auto Scaling responds to a scale\-out event, it launches one or more instances\. These instances start in the `Pending` state\. If you added an `autoscaling:EC2_INSTANCE_LAUNCHING` lifecycle hook to your Auto Scaling group, the instances move from the `Pending` state to the `Pending:Wait` state\. After you complete the lifecycle action, the instances enter the `Pending:Proceed` state\. When the instances are fully configured, they are attached to the Auto Scaling group and they enter the `InService` state\.

When Amazon EC2 Auto Scaling responds to a scale\-in event, it terminates one or more instances\. These instances are detached from the Auto Scaling group and enter the `Terminating` state\. If you added an `autoscaling:EC2_INSTANCE_TERMINATING` lifecycle hook to your Auto Scaling group, the instances move from the `Terminating` state to the `Terminating:Wait` state\. After you complete the lifecycle action, the instances enter the `Terminating:Proceed` state\. When the instances are fully terminated, they enter the `Terminated` state\.

For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

## Enter and exit standby<a name="as-lifecycle-standby"></a>

You can put any instance that is in an `InService` state into a `Standby` state\. This enables you to remove the instance from service, troubleshoot or make changes to it, and then put it back into service\.

Instances in a `Standby` state continue to be managed by the Auto Scaling group\. However, they are not an active part of your application until you put them back into service\.

For more information, see [Temporarily removing instances from your Auto Scaling group](as-enter-exit-standby.md)\.