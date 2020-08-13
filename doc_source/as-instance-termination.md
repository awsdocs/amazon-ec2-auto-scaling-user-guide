# Controlling which Auto Scaling instances terminate during scale in<a name="as-instance-termination"></a>

With each Auto Scaling group, you can control when it adds instances \(referred to as *scaling out*\) or removes instances \(referred to as *scaling in*\) from your network architecture\. You can scale the size of your group manually by adjusting your desired capacity, or you can automate the process through the use of scheduled scaling or a scaling policy\.

This topic describes the default termination policy and the options available to you to configure your own customized termination policies\. Using termination policies, you can control which instances you prefer to terminate first when a scale\-in event occurs\. 

It also describes how to enable instance scale\-in protection to prevent specific instances from being terminated during automatic scale in\. For instances in an Auto Scaling group, use Amazon EC2 Auto Scaling features to protect an instance when a scale\-in event occurs\. If you want to protect your instance from being accidentally terminated, use Amazon EC2 termination protection\. 

Note the following about Auto Scaling groups with a [mixed instances policy](asg-purchase-options.md):
+ Amazon EC2 Auto Scaling first identifies which of the two types \(Spot or On\-Demand\) should be terminated\. It then applies the termination policy in each Availability Zone individually, and identifies which instance \(within the identified purchase option\) in which Availability Zone to terminate that will result in the Availability Zones being most balanced\. The same principles apply to Auto Scaling groups that use a mixed instances configuration with weights defined for the instance types\. 

**Contents**
+ [Default termination policy](#default-termination-policy)
+ [Customizing the termination policy](#custom-termination-policy)
+ [Instance scale\-in protection](#instance-protection)
  + [Enable instance scale\-in protection for a group](#instance-protection-group)
  + [Modify the instance scale\-in protection setting for a group](#instance-protection-modify)
  + [Modify the instance scale\-in protection setting for an instance](#instance-protection-instance)
+ [Common termination policy scenarios for Amazon EC2 Auto Scaling](common-scenarios-termination.md)
  + [Scale\-in events](common-scenarios-termination.md#common-scenarios-termination-scale-in)
  + [Rebalancing activities](common-scenarios-termination.md#common-scenarios-termination-rebalancing)
    + [Availability outage](common-scenarios-termination.md#common-scenarios-termination-outage)
    + [Changes to Availability Zones](common-scenarios-termination.md#common-scenarios-termination-new-zone)
    + [Removing instances](common-scenarios-termination.md#common-scenarios-termination-removed-instances)
  + [Instance refreshes](common-scenarios-termination.md#common-scenarios-termination-instance-types)

## Default termination policy<a name="default-termination-policy"></a>

The default termination policy is designed to help ensure that your instances [span Availability Zones evenly for high availability](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\. The default policy is kept generic and flexible to cover a range of scenarios\. 

Before Amazon EC2 Auto Scaling selects an instance to terminate, it first determines which Availability Zones have the most instances, and at least one instance that is not protected from scale in\.

Within the selected Availability Zone, the default termination policy behavior is as follows:

1. Determine which instances to terminate so as to align the remaining instances to the allocation strategy for the On\-Demand or Spot Instance that is terminating\. This only applies to an Auto Scaling group that specifies a mixed instances policy, which uses [allocation strategies](asg-purchase-options.md#asg-allocation-strategies)\.

   For example, after your instances launch, you change the priority order of your preferred instance types\. When a scale\-in event occurs, Amazon EC2 Auto Scaling tries to gradually shift the On\-Demand Instances away from instance types that are lower priority\.

1. Determine whether any of the instances use the oldest launch template or configuration:

   1. \[For Auto Scaling groups that use a launch template\]

      Determine whether any of the instances use the oldest launch template unless there are instances that use a launch configuration\. Amazon EC2 Auto Scaling terminates instances that use a launch configuration before instances that use a launch template\.

   1. \[For Auto Scaling groups that use a launch configuration\]

      Determine whether any of the instances use the oldest launch configuration\. 

1. After applying all of the above criteria, if there are multiple unprotected instances to terminate, determine which instances are closest to the next billing hour\. If there are multiple unprotected instances closest to the next billing hour, terminate one of these instances at random\.

   Note that terminating the instance closest to the next billing hour helps you maximize the use of your instances that have an hourly charge\. Alternatively, if your Auto Scaling group uses Amazon Linux or Ubuntu, your EC2 usage is billed in one\-second increments\. For more information, see [Amazon EC2 pricing](https://aws.amazon.com/ec2/pricing/)\.

## Customizing the termination policy<a name="custom-termination-policy"></a>

You have the option of replacing the default policy with a customized one to support common use cases like keeping instances that have the desired version of your application\. 

When you customize the termination policy, if one Availability Zone has more instances than the other Availability Zones that are used by the group, your termination policy is applied to the instances from the imbalanced Availability Zone\. If the Availability Zones used by the group are balanced, the termination policy is applied across all of the Availability Zones for the group\.

Amazon EC2 Auto Scaling supports the following termination policies:
+ `Default`\. Terminate instances according to the default termination policy\. This policy is useful when you want your Spot allocation strategy evaluated before any other policy, so that every time your Spot instances are terminated or replaced, you continue to make use of Spot Instances in the optimal pools\. It is also useful, for example, when you want to move off launch configurations and start using launch templates\.
+ `AllocationStrategy`\. Terminate instances in the Auto Scaling group to align the remaining instances to the allocation strategy for the type of instance that is terminating \(either a Spot Instance or an On\-Demand Instance\)\. This policy is useful when your preferred instance types have changed\. If the Spot allocation strategy is `lowest-price`, you can gradually rebalance the distribution of Spot Instances across your N lowest priced Spot pools\. If the Spot allocation strategy is `capacity-optimized`, you can gradually rebalance the distribution of Spot Instances across Spot pools where there is more available Spot capacity\. You can also gradually replace On\-Demand Instances of a lower priority type with On\-Demand Instances of a higher priority type\.
+ `OldestLaunchTemplate`\. Terminate instances that have the oldest launch template\. With this policy, instances that use the noncurrent launch template are terminated first, followed by instances that use the oldest version of the current launch template\. This policy is useful when you're updating a group and phasing out the instances from a previous configuration\.
+ `OldestLaunchConfiguration`\. Terminate instances that have the oldest launch configuration\. This policy is useful when you're updating a group and phasing out the instances from a previous configuration\.
+ `ClosestToNextInstanceHour`\. Terminate instances that are closest to the next billing hour\. This policy helps you maximize the use of your instances that have an hourly charge\.
+ `NewestInstance`\. Terminate the newest instance in the group\. This policy is useful when you're testing a new launch configuration but don't want to keep it in production\.
+ `OldestInstance`\. Terminate the oldest instance in the group\. This option is useful when you're upgrading the instances in the Auto Scaling group to a new EC2 instance type\. You can gradually replace instances of the old type with instances of the new type\.
**Note**  
Amazon EC2 Auto Scaling always balances instances across Availability Zones first, regardless of which termination policy is used\. As a result, you might encounter situations in which some newer instances are terminated before older instances when there is a more recently added Availability Zone, or when one Availability Zone has more instances than the other Availability Zones that are used by the group\.

**To customize a termination policy \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\. \(Old console: On the **Details** tab, choose **Edit**\.\)

1. For **Termination policies**, choose one or more termination policies\. If you choose multiple policies, list them in the order in which they should apply\. If you use the **Default** policy, make it the last one in the list\.

1. Choose **Update**\.

**To customize a termination policy \(AWS CLI\)**  
Use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html)
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html)

You can use these policies individually, or combine them into a list of policies\. For example, use the following command to update an Auto Scaling group to use the `OldestLaunchConfiguration` policy first and then use the `ClosestToNextInstanceHour` policy\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --termination-policies "OldestLaunchConfiguration" "ClosestToNextInstanceHour"
```

If you use the `Default` termination policy, make it the last one in the list of termination policies\. For example, `--termination-policies "OldestLaunchConfiguration" "Default"`\.

## Instance scale\-in protection<a name="instance-protection"></a>

To control whether an Auto Scaling group can terminate a particular instance when scaling in, use instance scale\-in protection\. You can enable the instance scale\-in protection setting on an Auto Scaling group or on an individual Auto Scaling instance\. When the Auto Scaling group launches an instance, it inherits the instance scale\-in protection setting of the Auto Scaling group\. You can change the instance scale\-in protection setting for an Auto Scaling group or an Auto Scaling instance at any time\.

Instance scale\-in protection starts when the instance state is `InService`\. If you detach an instance that is protected from termination, its instance scale\-in protection setting is lost\. When you attach the instance to the group again, it inherits the current instance scale\-in protection setting of the group\.

If all instances in an Auto Scaling group are protected from termination during scale in, and a scale\-in event occurs, its desired capacity is decremented\. However, the Auto Scaling group can't terminate the required number of instances until their instance scale\-in protection settings are disabled\.

Instance scale\-in protection does not protect Auto Scaling instances from the following:
+ Manual termination through the Amazon EC2 console, the `terminate-instances` command, or the `TerminateInstances` action\. To protect Auto Scaling instances from manual termination, enable Amazon EC2 termination protection\. For more information, see [Enabling termination protection](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingDisableAPITermination) in the *Amazon EC2 User Guide for Linux Instances*\.
+ Health check replacement if the instance fails health checks\. For more information, see [Health checks for Auto Scaling instances](healthcheck.md)\. To prevent Amazon EC2 Auto Scaling from terminating unhealthy instances, suspend the `ReplaceUnhealthy` process\. For more information, see [Suspending and resuming scaling processes](as-suspend-resume-processes.md)\.
+ Spot Instance interruptions\. A Spot Instance is terminated when capacity is no longer available or the Spot price exceeds your maximum price\. 

**Topics**
+ [Enable instance scale\-in protection for a group](#instance-protection-group)
+ [Modify the instance scale\-in protection setting for a group](#instance-protection-modify)
+ [Modify the instance scale\-in protection setting for an instance](#instance-protection-instance)

### Enable instance scale\-in protection for a group<a name="instance-protection-group"></a>

You can enable instance scale\-in protection when you create an Auto Scaling group\. By default, instance scale\-in protection is disabled\.

**To enable instance scale\-in protection \(new console\)**  
When you create the Auto Scaling group, on the **Configure group size and scaling policies** page, under **Instance scale\-in protection**, select the **Enable instance scale\-in protection** option\.

**To enable instance scale\-in protection \(old console\)**  
When you create the Auto Scaling group, on the **Configure Auto Scaling group details** page, under **Advanced Details**, select the `Protect From Scale In` option from **Instance Protection**\.

**To enable instance scale\-in protection \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to enable instance scale\-in protection\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg --new-instances-protected-from-scale-in ...
```

### Modify the instance scale\-in protection setting for a group<a name="instance-protection-modify"></a>

You can enable or disable the instance scale\-in protection setting for an Auto Scaling group\. When the instance scale\-in protection setting is enabled, all new instances launched after enabling it will have instance scale\-in protection enabled\. Previously launched instances are not protected from scale in unless you enable the instance scale\-in protection setting for each instance individually\.

**To change the instance scale\-in protection setting for a group \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select check box next to the Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\. \(Old console: On the **Details** tab, choose **Edit**\.\)

1. For **Instance scale\-in protection**, select **Enable instance scale\-in protection**\. \(Old console: For **Instance Protection**, select **Protect From Scale In**\.\) 

1. Choose **Update**\.

**To change the instance scale\-in protection setting for a group \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to enable instance scale\-in protection for the specified Auto Scaling group\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --new-instances-protected-from-scale-in
```

Use the following command to disable instance scale\-in protection for the specified group\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --no-new-instances-protected-from-scale-in
```

### Modify the instance scale\-in protection setting for an instance<a name="instance-protection-instance"></a>

By default, an instance gets its instance scale\-in protection setting from its Auto Scaling group\. However, you can enable or disable instance scale\-in protection for an instance at any time\.

**To change the instance scale\-in protection setting for an instance \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Instance management** tab, in **Instances**, select an instance\. \(Old console: The **Instances** tab is where you can select the instance\.\) 

1. To enable instance scale\-in protection, choose **Actions**, **Set scale\-in protection**\. When prompted, choose **Set scale\-in protection**\.

1. To disable instance scale\-in protection, choose **Actions**, **Remove scale\-in protection**\. When prompted, choose **Remove scale\-in protection**\.

**To change the instance scale\-in protection setting for an instance \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-protection.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-protection.html) command to enable instance scale\-in protection for the specified instance\.

```
aws autoscaling set-instance-protection --instance-ids i-5f2e8a0d --auto-scaling-group-name my-asg --protected-from-scale-in
```

Use the following command to disable instance scale\-in protection for the specified instance\.

```
aws autoscaling set-instance-protection --instance-ids i-5f2e8a0d --auto-scaling-group-name my-asg --no-protected-from-scale-in
```