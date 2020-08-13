# Suspending and resuming scaling processes<a name="as-suspend-resume-processes"></a>

This topic explains how to suspend and then resume one or more of the scaling processes for your Auto Scaling group\. It also describes the issues to consider when choosing to use the suspend\-resume feature of Amazon EC2 Auto Scaling\.

**Important**  
Use the standby feature instead of the suspend\-resume feature if you need to troubleshoot or reboot an instance\. For more information, see [Temporarily removing instances from your Auto Scaling group](as-enter-exit-standby.md)\. Use the instance scale\-in protection feature to prevent specific instances from being terminated during automatic scale in\. For more information, see [Instance scale\-in protection](as-instance-termination.md#instance-protection)\.

In addition to suspensions that you initiate, Amazon EC2 Auto Scaling can also suspend processes for Auto Scaling groups that repeatedly fail to launch instances\. This is known as an *administrative suspension*\. An administrative suspension most commonly applies to Auto Scaling groups that have been trying to launch instances for over 24 hours but have not succeeded in launching any instances\. You can resume processes that were suspended by Amazon EC2 Auto Scaling for administrative reasons\.

**Topics**
+ [Scaling processes](#process-types)
+ [Choosing to suspend](#choosing-suspend-resume)
+ [Suspend and resume scaling processes \(console\)](#as-suspend-resume)
+ [Suspend and resume scaling processes \(AWS CLI\)](#as-suspend-resume-aws-cli)

## Scaling processes<a name="process-types"></a>

For Amazon EC2 Auto Scaling, there are two primary process types: `Launch` and `Terminate`\. The `Launch` process adds a new Amazon EC2 instance to an Auto Scaling group, increasing its capacity\. The `Terminate` process removes an Amazon EC2 instance from the group, decreasing its capacity\. 

The other process types for Amazon EC2 Auto Scaling relate to specific scaling features: 
+ `AddToLoadBalancer`—Adds instances to the attached load balancer or target group when they are launched\.
+ `AlarmNotification`—Accepts notifications from CloudWatch alarms that are associated with the group's scaling policies\.
+ `AZRebalance`—Balances the number of EC2 instances in the group evenly across all of the specified Availability Zones when the group becomes unbalanced, for example, a previously unavailable Availability Zone returns to a healthy state\. For more information, see [Rebalancing activities](auto-scaling-benefits.md#AutoScalingBehavior.InstanceUsage)\.
+ `HealthCheck`—Checks the health of the instances and marks an instance as unhealthy if Amazon EC2 or Elastic Load Balancing tells Amazon EC2 Auto Scaling that the instance is unhealthy\. This process can override the health status of an instance that you set manually\. For more information, see [Health checks for Auto Scaling instances](healthcheck.md)\.
+ `ReplaceUnhealthy`—Terminates instances that are marked as unhealthy and then creates new instances to replace them\.
+ `ScheduledActions`—Performs the scheduled scaling actions that you create or that are created by the predictive scaling feature of AWS Auto Scaling\. 

## Choosing to suspend<a name="choosing-suspend-resume"></a>

Each process type can be suspended and resumed independently\. This section provides some guidance and behavior to take into account before deciding to suspend a scaling process\. Keep in mind that suspending individual processes might interfere with other processes\. Depending on the reason for suspending a process, you might need to suspend multiple processes together\. 

The following descriptions explain what happens when individual process types are suspended\. 

**Warning**  
If you suspend either the `Launch` or `Terminate` process types, it can prevent other process types from functioning properly\.

`Terminate`
+ Your Auto Scaling group does not scale in for alarms or scheduled actions that occur while the process is suspended\. In addition, the following processes are disrupted:
  + `AZRebalance` is still active but does not function properly\. It can launch new instances without terminating the old ones\. This could cause your Auto Scaling group to grow up to 10 percent larger than its maximum size, because this is allowed temporarily during rebalancing activities\. Your Auto Scaling group could remain above its maximum size until you resume the `Terminate` process\. When `Terminate` resumes, `AZRebalance` gradually rebalances the Auto Scaling group if the group is no longer balanced between Availability Zones or if different Availability Zones are specified\.
  + `ReplaceUnhealthy` is inactive but not `HealthCheck`\. When `Terminate` resumes, the `ReplaceUnhealthy` process immediately starts running\. If any instances were marked as unhealthy while `Terminate` was suspended, they are immediately replaced\.

`Launch`
+ Your Auto Scaling group does not scale out for alarms or scheduled actions that occur while the process is suspended\. `AZRebalance` stops rebalancing the group\. `ReplaceUnhealthy` continues to terminate unhealthy instances, but does not launch replacements\. When you resume `Launch`, rebalancing activities and health check replacements are handled in the following way: 
  + `AZRebalance` gradually rebalances the Auto Scaling group if the group is no longer balanced between Availability Zones or if different Availability Zones are specified\.
  + `ReplaceUnhealthy` immediately replaces any instances that it terminated during the time that `Launch` was suspended\.

`AddToLoadBalancer`
+ Amazon EC2 Auto Scaling launches the instances but does not add them to the load balancer or target group\. When you resume the `AddToLoadBalancer` process, it resumes adding instances to the load balancer or target group when they are launched\. However, it does not add the instances that were launched while this process was suspended\. You must register those instances manually\.

`AlarmNotification`
+ Amazon EC2 Auto Scaling does not execute scaling policies when a CloudWatch alarm threshold is in breach\. Suspending `AlarmNotification` allows you to temporarily stop scaling events triggered by the group's scaling policies without deleting the scaling policies or their associated CloudWatch alarms\. When you resume `AlarmNotification`, Amazon EC2 Auto Scaling considers policies with alarm thresholds that are currently in breach\.

`AZRebalance`
+ Your Auto Scaling group does not attempt to redistribute instances after certain events\. However, if a scale\-out or scale\-in event occurs, the scaling process still tries to balance the Availability Zones\. For example, during scale out, it launches the instance in the Availability Zone with the fewest instances\. If the group becomes unbalanced while `AZRebalance` is suspended and you resume it, Amazon EC2 Auto Scaling attempts to rebalance the group\. It first calls `Launch` and then `Terminate`\.

`HealthCheck`
+ Amazon EC2 Auto Scaling stops marking instances unhealthy as a result of EC2 and Elastic Load Balancing health checks\. Your custom health checks continue to function properly, however\. After you suspend `HealthCheck`, if you need to, you can manually set the health state of instances in your group and have `ReplaceUnhealthy` replace them\.

`ReplaceUnhealthy`
+ Amazon EC2 Auto Scaling stops replacing instances that are marked as unhealthy\. Instances that fail EC2 or Elastic Load Balancing health checks are still marked as unhealthy\. As soon as you resume the `ReplaceUnhealthly` process, Amazon EC2 Auto Scaling replaces instances that were marked unhealthy while this process was suspended\. The `ReplaceUnhealthy` process calls both of the primary process types—first `Terminate` and then `Launch`\. 

`ScheduledActions`
+ Amazon EC2 Auto Scaling does not execute scaling actions that are scheduled to run during the suspension period\. When you resume `ScheduledActions`, Amazon EC2 Auto Scaling only considers scheduled actions whose execution time has not yet passed\. 

### Suspending both launch and terminate<a name="suspend-launch-terminate"></a>

When you suspend the `Launch` and `Terminate` process types together, the following happens: 
+ Your Auto Scaling group cannot initiate scaling activities or maintain its desired capacity\. 
+ If the group becomes unbalanced between Availability Zones, Amazon EC2 Auto Scaling does not attempt to redistribute instances evenly between the Availability Zones that are specified for your Auto Scaling group\.
+ Your Auto Scaling group cannot replace instances that are marked unhealthy\. 

When you resume the `Launch` and `Terminate` process types, Amazon EC2 Auto Scaling replaces instances that were marked unhealthy while the processes were suspended and might attempt to rebalance the group\. Scaling activities also resume\. 

### Additional considerations<a name="other-considerations"></a>

There are some outside operations that might be affected while `Launch` and `Terminate` are suspended\.
+ **Spot Instance Interruptions**—If `Terminate` is suspended and your Auto Scaling group has Spot Instances, they can still terminate in the event that Spot capacity is no longer available\. While `Launch` is suspended, Amazon EC2 Auto Scaling cannot launch replacement instances from another Spot Instance pool or from the same Spot Instance pool when it is available again\.
+ **Attaching and Detaching Instances **—When `Launch` and `Terminate` are suspended, you can detach instances that are attached to your Auto Scaling group, but you can't attach new instances to the group\. To attach instances, you must first resume `Launch`\. 
**Note**  
If detaching an instance will immediately be followed by manually terminating it, you can call the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/terminate-instance-in-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/terminate-instance-in-auto-scaling-group.html) CLI command instead\. This terminates the specified instance and optionally adjusts the group's desired capacity\. In addition, if the Auto Scaling group is being used with lifecycle hooks, the custom actions that you specified for instance termination will run before the instance is fully terminated\.
+ **Standby Instances**—While `Launch` is suspended, you cannot return an instance in the `Standby` state to service\. To return the instance to service, you must first resume `Launch`\. 

## Suspend and resume scaling processes \(console\)<a name="as-suspend-resume"></a>

You can suspend and resume individual processes or all processes\.

**To suspend and resume processes**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\. \(Old console: On the **Details** tab, choose **Edit**\.\)

1. For **Suspended processes**, choose the process to suspend\.

   To resume a suspended process, remove it from **Suspended processes**\.

1. Choose **Update**\.

## Suspend and resume scaling processes \(AWS CLI\)<a name="as-suspend-resume-aws-cli"></a>

You can suspend and resume individual processes or all processes\.

**To suspend a process**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/suspend-processes.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/suspend-processes.html) command with the `--scaling-processes` option as follows\.

```
aws autoscaling suspend-processes --auto-scaling-group-name my-asg --scaling-processes AlarmNotification
```

**To suspend all processes**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/suspend-processes.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/suspend-processes.html) command as follows \(omitting the `--scaling-processes` option\)\.

```
aws autoscaling suspend-processes --auto-scaling-group-name my-asg
```

**To resume a suspended process**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/resume-processes.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/resume-processes.html) command as follows\.

```
aws autoscaling resume-processes --auto-scaling-group-name my-asg --scaling-processes AlarmNotification
```

**To resume all suspended processes**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/resume-processes.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/resume-processes.html) command as follows \(omitting the `--scaling-processes` option\)\.

```
aws autoscaling resume-processes --auto-scaling-group-name my-asg
```