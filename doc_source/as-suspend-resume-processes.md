# Suspend and resume a process for an Auto Scaling group<a name="as-suspend-resume-processes"></a>

This topic explains how to suspend and then resume one or more of the processes for your Auto Scaling group\. You might want to do this, for example, so that you can investigate a configuration issue that is causing the process to fail, or to prevent Amazon EC2 Auto Scaling from marking instances unhealthy and replacing them while you are making changes to your Auto Scaling group\.

**Topics**
+ [Types of processes](#process-types)
+ [Considerations](#suspend-resume-considerations)
+ [Suspend and resume processes \(console\)](#as-suspend-resume)
+ [Suspend and resume processes \(AWS CLI\)](#as-suspend-resume-aws-cli)

**Note**  
In addition to suspensions that you initiate, Amazon EC2 Auto Scaling can also suspend processes for Auto Scaling groups that repeatedly fail to launch instances\. This is known as an *administrative suspension*\. An administrative suspension most commonly applies to Auto Scaling groups that have been trying to launch instances for over 24 hours but have not succeeded in launching any instances\. You can resume processes that were suspended by Amazon EC2 Auto Scaling for administrative reasons\.

## Types of processes<a name="process-types"></a>

The suspend\-resume feature supports the following processes:
+ `Launch`—Adds instances to the Auto Scaling group when the group scales out, or when Amazon EC2 Auto Scaling chooses to launch instances for other reasons, such as when it adds instances to a warm pool\.
+ `Terminate`—Removes instances from the Auto Scaling group when the group scales in, or when Amazon EC2 Auto Scaling chooses to terminate instances for other reasons, such as when an instance is terminated for exceeding its maximum lifetime duration or failing a health check\.
+ `AddToLoadBalancer`—Adds instances to the attached load balancer target group or Classic Load Balancer when they are launched\. For more information, see [Use Elastic Load Balancing to distribute traffic across the instances in your Auto Scaling group](autoscaling-load-balancer.md)\.
+ `AlarmNotification`—Accepts notifications from CloudWatch alarms that are associated with dynamic scaling policies\. For more information, see [Dynamic scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)\.
+ `AZRebalance`—Balances the number of EC2 instances in the group evenly across all of the specified Availability Zones when the group becomes unbalanced, for example, when a previously unavailable Availability Zone returns to a healthy state\. For more information, see [Rebalancing activities](auto-scaling-benefits.md#AutoScalingBehavior.InstanceUsage)\.
+ `HealthCheck`—Checks the health of the instances and marks an instance as unhealthy if Amazon EC2 or Elastic Load Balancing tells Amazon EC2 Auto Scaling that the instance is unhealthy\. This process can override the health status of an instance that you set manually\. For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\.
+ `InstanceRefresh`—Terminates and replaces instances using the instance refresh feature\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.
+ `ReplaceUnhealthy`—Terminates instances that are marked as unhealthy and then creates new instances to replace them\. For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\.
+ `ScheduledActions`—Performs the scheduled scaling actions that you create or that are created for you when you create an AWS Auto Scaling scaling plan and turn on predictive scaling\. For more information, see [Scheduled scaling for Amazon EC2 Auto Scaling](ec2-auto-scaling-scheduled-scaling.md)\. 

## Considerations<a name="suspend-resume-considerations"></a>

Consider the following before suspending processes:
+ You can suspend and resume individual processes or all processes\.
+ Suspending a process affects all instances in your Auto Scaling group\. For example, you can suspend the `HealthCheck` and `ReplaceUnhealthy` processes to reboot instances without Amazon EC2 Auto Scaling terminating the instances based on its health checks\. If you need Amazon EC2 Auto Scaling to perform health checks on remaining instances, then use the standby feature instead of the suspend\-resume feature\. For more information, see [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md)\.
+ Suspending `AlarmNotification` allows you to temporarily stop the group's target tracking, step, and simple scaling policies without deleting the scaling policies or their associated CloudWatch alarms\. To temporarily stop individual scaling policies instead, see [Disable a scaling policy for an Auto Scaling group](as-enable-disable-scaling-policy.md)\.
+ If you suspend the `Launch` and `Terminate` processes, or `AZRebalance`, and then you make changes to your Auto Scaling group, for example, by detaching instances or changing the Availability Zones that are specified, your group can become unbalanced between Availability Zones\. If that happens, after you resume the suspended processes, Amazon EC2 Auto Scaling gradually redistributes instances evenly between the Availability Zones\.
+ Suspending the `Terminate` process doesn't prevent the successful termination of instances using the force delete option with the [delete\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html) command\.
+ The `RemoveFromLoadBalancerLowPriority` process should be ignored when it is present in calls to describe Auto Scaling groups using the AWS CLI or SDKs\. This process is deprecated and is retained only for backward compatibility\. 

### Understand how suspending processes affects other processes<a name="understand-how-suspending-processes-affects-other-processes"></a>

The following descriptions explain what happens when individual process types are suspended\. 

Scenario 1: `Launch` is suspended
+ `AlarmNotification` is still active, but your Auto Scaling group can't initiate scale\-out activities for alarms that are in breach\. 
+ `ScheduledActions` is active, but your Auto Scaling group can't initiate scale\-out activities for any scheduled actions that occur\. 
+ `AZRebalance` stops rebalancing the group\.
+ `ReplaceUnhealthy` continues to terminate unhealthy instances, but does not launch replacements\. When you resume the `Launch` process, Amazon EC2 Auto Scaling immediately replaces any instances that it terminated during the time that `Launch` was suspended\.
+ `InstanceRefresh` does not replace instances\.

Scenario 2: `Terminate` is suspended
+ `AlarmNotification` is still active, but your Auto Scaling group can't initiate scale\-in activities for alarms that are in breach\. 
+ `ScheduledActions` is active, but your Auto Scaling group can't initiate scale\-in activities for any scheduled actions that occur\. 
+ `AZRebalance` is still active but does not function properly\. It can launch new instances without terminating the old ones\. This could cause your Auto Scaling group to grow up to 10 percent larger than its maximum size, because this is allowed temporarily during rebalancing activities\. Your Auto Scaling group could remain above its maximum size until you resume the `Terminate` process\.
+ `ReplaceUnhealthy` is inactive but not `HealthCheck`\. When `Terminate` resumes, the `ReplaceUnhealthy` process immediately starts running\. If any instances were marked as unhealthy while `Terminate` was suspended, they are immediately replaced\.
+ `InstanceRefresh` does not replace instances\.

Scenario 3: `AddToLoadBalancer` is suspended
+ Amazon EC2 Auto Scaling launches the instances but does not add them to the load balancer target group or Classic Load Balancer\. When you resume the `AddToLoadBalancer` process, it resumes adding instances to the load balancer when they are launched\. However, it does not add the instances that were launched while this process was suspended\. You must register those instances manually\.

Scenario 4: `AlarmNotification` is suspended
+ Amazon EC2 Auto Scaling does not invoke scaling policies when a CloudWatch alarm threshold is in breach\. When you resume `AlarmNotification`, Amazon EC2 Auto Scaling considers policies with alarm thresholds that are currently in breach\.

Scenario 5: `AZRebalance` is suspended
+ Amazon EC2 Auto Scaling does not attempt to redistribute instances after certain events\. However, if a scale\-out or scale\-in event occurs, the scaling process still tries to balance the Availability Zones\. For example, during scale out, it launches the instance in the Availability Zone with the fewest instances\. If the group becomes unbalanced while `AZRebalance` is suspended and you resume it, Amazon EC2 Auto Scaling attempts to rebalance the group\. It first calls `Launch` and then `Terminate`\.

Scenario 6: `HealthCheck` is suspended
+ Amazon EC2 Auto Scaling stops marking instances unhealthy as a result of EC2 and Elastic Load Balancing health checks\. Your custom health checks continue to function properly\. After you suspend `HealthCheck`, if you need to, you can manually set the health state of instances in your group and have `ReplaceUnhealthy` replace them\.

Scenario 7: `InstanceRefresh` is suspended
+ Amazon EC2 Auto Scaling stops replacing instances as a result of an instance refresh\. If there is an instance refresh in progress, this pauses the operation without canceling it\.

Scenario 8: `ReplaceUnhealthy` is suspended
+ Amazon EC2 Auto Scaling stops replacing instances that are marked as unhealthy\. Instances that fail EC2 or Elastic Load Balancing health checks are still marked as unhealthy\. As soon as you resume the `ReplaceUnhealthy` process, Amazon EC2 Auto Scaling replaces instances that were marked unhealthy while this process was suspended\. The `ReplaceUnhealthy` process calls `Terminate` first and then `Launch`\. 

Scenario 9: `ScheduledActions` is suspended
+ Amazon EC2 Auto Scaling does not run scheduled actions that are scheduled to run during the suspension period\. When you resume `ScheduledActions`, Amazon EC2 Auto Scaling only considers scheduled actions whose scheduled time has not yet passed\. 

#### Additional considerations<a name="other-considerations"></a>

In addition, when `Launch` or `Terminate` are suspended, the following features might not function correctly:
+ **Maximum instance lifetime**—When `Launch` or `Terminate` are suspended, the maximum instance lifetime feature can't replace any instances\. 
+ **Spot Instance interruptions**—If `Terminate` is suspended and your Auto Scaling group has Spot Instances, they can still terminate in the event that Spot capacity is no longer available\. While `Launch` is suspended, Amazon EC2 Auto Scaling can't launch replacement instances from another Spot Instance pool or from the same Spot Instance pool when it is available again\.
+ **Capacity Rebalancing**—If `Terminate` is suspended and you use Capacity Rebalancing to handle Spot Instance interruptions, the Amazon EC2 Spot service can still terminate instances in the event that Spot capacity is no longer available\. If `Launch` is suspended, Amazon EC2 Auto Scaling can't launch replacement instances from another Spot Instance pool or from the same Spot Instance pool when it is available again\.
+ **Attaching and detaching instances **—When `Launch` and `Terminate` are suspended, you can detach instances that are attached to your Auto Scaling group, but while `Launch` is suspended, you can't attach new instances to the group\. 
+ **Standby instances**—When `Launch` and `Terminate` are suspended, you can put an instance in the `Standby` state, but while `Launch` is suspended, you can't return an instance in the `Standby` state to service\. 

## Suspend and resume processes \(console\)<a name="as-suspend-resume"></a>

Use the following procedure to suspend a process\.

**To suspend a process**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\.

1. For **Suspended processes**, choose the process to suspend\.

1. Choose **Update**\.

When you are ready, use the following procedure to resume the suspended process\.

**To resume a process**

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\.

1. For **Suspended processes**, remove the suspended process\.

1. Choose **Update**\.

## Suspend and resume processes \(AWS CLI\)<a name="as-suspend-resume-aws-cli"></a>

Use the following [suspend\-processes](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/suspend-processes.html) command to suspend individual processes\.

```
aws autoscaling suspend-processes --auto-scaling-group-name my-asg --scaling-processes HealthCheck ReplaceUnhealthy 
```

To suspend all processes, omit the `--scaling-processes` option, as follows\. 

```
aws autoscaling suspend-processes --auto-scaling-group-name my-asg
```

When you are ready to resume a suspended process, use the following [resume\-processes](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/resume-processes.html) command\. 

```
aws autoscaling resume-processes --auto-scaling-group-name my-asg --scaling-processes HealthCheck
```

To resume all suspended processes, omit the `--scaling-processes` option, as follows\.

```
aws autoscaling resume-processes --auto-scaling-group-name my-asg
```