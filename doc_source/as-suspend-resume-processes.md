# Suspending and Resuming Scaling Processes<a name="as-suspend-resume-processes"></a>

You can suspend and then resume one or more of the scaling processes for your Auto Scaling group\. This can be useful when you want to investigate a configuration problem or other issue with your web application and then make changes to your application, without invoking the scaling processes\.

Amazon EC2 Auto Scaling can suspend processes for Auto Scaling groups that repeatedly fail to launch instances\. This is known as an *administrative suspension*, and most commonly applies to Auto Scaling groups that have been trying to launch instances for over 24 hours but have not succeeded in launching any instances\. You can resume processes suspended for administrative reasons\.

**Topics**
+ [Scaling Processes](#process-types)
+ [Suspend and Resume Processes Using the Console](#as-suspend-resume)
+ [Suspend and Resume Processes Using the AWS CLI](#as-suspend-resume-aws-cli)

## Scaling Processes<a name="process-types"></a>

Amazon EC2 Auto Scaling supports the following scaling processes:

`Launch`  
Adds a new EC2 instance to the group, increasing its capacity\.  
If you suspend `Launch`, this disrupts other processes\. For example, you can't return an instance in a standby state to service if the `Launch` process is suspended, because the group can't scale\.

`Terminate`  
Removes an EC2 instance from the group, decreasing its capacity\.  
If you suspend `Terminate`, this disrupts other processes\.

`HealthCheck`  
Checks the health of the instances\. Amazon EC2 Auto Scaling marks an instance as unhealthy if Amazon EC2 or Elastic Load Balancing tells Amazon EC2 Auto Scaling that the instance is unhealthy\. This process can override the health status of an instance that you set manually\.

`ReplaceUnhealthy`  
Terminates instances that are marked as unhealthy and later creates new instances to replace them\. This process works with the `HealthCheck` process, and uses both the `Terminate` and `Launch` processes\.

`AZRebalance`  
Balances the number of EC2 instances in the group across the Availability Zones in the region\. If you remove an Availability Zone from your Auto Scaling group or an Availability Zone otherwise becomes unhealthy or unavailable, the scaling process launches new instances in an unaffected Availability Zone before terminating the unhealthy or unavailable instances\. When the unhealthy Availability Zone returns to a healthy state, the scaling process automatically redistributes the instances evenly across the Availability Zones for the group\. For more information, see [Rebalancing Activities](auto-scaling-benefits.md#AutoScalingBehavior.InstanceUsage)\.  
If you suspend `AZRebalance` and a scale\-out or scale\-in event occurs, the scaling process still tries to balance the Availability Zones\. For example, during scale\-out, it launches the instance in the Availability Zone with the fewest instances\.  
If you suspend the `Launch` process, `AZRebalance` neither launches new instances nor terminates existing instances\. This is because `AZRebalance` terminates instances only after launching the replacement instances\. If you suspend the `Terminate` process, your Auto Scaling group can grow up to ten percent larger than its maximum size, because this is allowed temporarily during rebalancing activities\. If the scaling process cannot terminate instances, your Auto Scaling group could remain above its maximum size until you resume the `Terminate` process\.

`AlarmNotification`  
Accepts notifications from CloudWatch alarms that are associated with the group\.  
If you suspend `AlarmNotification`, Amazon EC2 Auto Scaling does not automatically execute policies that would be triggered by an alarm\. If you suspend `Launch` or `Terminate`, it will not be able to execute scale\-out or scale\-in policies, respectively\.

`ScheduledActions`  
Performs scheduled actions that you create\.  
If you suspend `Launch` or `Terminate`, scheduled actions that involve launching or terminating instances are affected\.

`AddToLoadBalancer`  
Adds instances to the attached load balancer or target group when they are launched\.  
If you suspend `AddToLoadBalancer`, Amazon EC2 Auto Scaling launches the instances but does not add them to the load balancer or target group\. If you resume the `AddToLoadBalancer` process, it resumes adding instances to the load balancer or target group when they are launched\. However, it does not add the instances that were launched while this process was suspended\. You must register those instances manually\.

## Suspend and Resume Processes Using the Console<a name="as-suspend-resume"></a>

You can suspend and resume individual processes using the AWS Management Console\.

**To suspend and resume processes using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select the Auto Scaling group\.

1. On the **Details** tab, choose **Edit**\.

1. For **Suspended Processes**, select the process to suspend\.  
![\[Suspended processes list.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-suspend-processes-add.png)

   To resume a suspended process, remove it from **Suspended Processes**\.  
![\[Suspended processes list.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-suspend-processes-remove.png)

1. Choose **Save**\.

## Suspend and Resume Processes Using the AWS CLI<a name="as-suspend-resume-aws-cli"></a>

You can suspend and resume individual processes or all processes\.

**To suspend a process**  
Use the [suspend\-processes](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/suspend-processes.html) command with the `--scaling-processes` option as follows:

```
aws autoscaling suspend-processes --auto-scaling-group-name my-asg --scaling-processes AlarmNotification
```

**To suspend all processes**  
Use the [suspend\-processes](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/suspend-processes.html) command as follows \(omitting the `--scaling-processes` option\):

```
aws autoscaling suspend-processes --auto-scaling-group-name my-asg
```

**To resume a suspended process**  
Use the [resume\-processes](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/resume-processes.html) command as follows:

```
aws autoscaling resume-processes --auto-scaling-group-name my-asg --scaling-processes AlarmNotification
```

**To resume all suspended processes**  
Use the [resume\-processes](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/resume-processes.html) command as follows \(omitting the `--scaling-processes` option\):

```
aws autoscaling resume-processes --auto-scaling-group-name my-asg
```