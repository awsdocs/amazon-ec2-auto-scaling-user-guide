# Using instance scale\-in protection<a name="ec2-auto-scaling-instance-protection"></a>

To control whether an Auto Scaling group can terminate a particular instance when scaling in, use instance scale\-in protection\. You can enable the instance scale\-in protection setting on an Auto Scaling group or on an individual Auto Scaling instance\. When the Auto Scaling group launches an instance, it inherits the instance scale\-in protection setting of the Auto Scaling group\. You can change the instance scale\-in protection setting for an Auto Scaling group or an Auto Scaling instance at any time\.

Instance scale\-in protection starts when the instance state is `InService`\. If you detach an instance that is protected from scale\-in, its instance scale\-in protection setting is lost\. When you attach the instance to the group again, it inherits the current instance scale\-in protection setting of the group\.

If all instances in an Auto Scaling group are protected from termination during scale in, and a scale\-in event occurs, its desired capacity is decremented\. However, the Auto Scaling group can't terminate the required number of instances until their instance scale\-in protection settings are disabled\.

Instance scale\-in protection does not protect Auto Scaling instances from the following:
+ Manual termination through the Amazon EC2 console, the `terminate-instances` command, or the `TerminateInstances` action\. To protect Auto Scaling instances from manual termination, enable Amazon EC2 termination protection\. For more information, see [Enabling termination protection](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingDisableAPITermination) in the *Amazon EC2 User Guide for Linux Instances*\.
+ Health check replacement if the instance fails health checks\. For more information, see [Health checks for Auto Scaling instances](healthcheck.md)\. To prevent Amazon EC2 Auto Scaling from terminating unhealthy instances, suspend the `ReplaceUnhealthy` process\. For more information, see [Suspending and resuming a process for an Auto Scaling group](as-suspend-resume-processes.md)\.
+ Spot Instance interruptions\. A Spot Instance is terminated when capacity is no longer available or the Spot price exceeds your maximum price\. 

**Topics**
+ [Enable instance scale\-in protection for a group](#instance-protection-group)
+ [Modify the instance scale\-in protection setting for a group](#instance-protection-modify)
+ [Modify the instance scale\-in protection setting for an instance](#instance-protection-instance)

## Enable instance scale\-in protection for a group<a name="instance-protection-group"></a>

You can enable instance scale\-in protection when you create an Auto Scaling group\. By default, instance scale\-in protection is disabled\.

**To enable instance scale\-in protection \(console\)**  
When you create the Auto Scaling group, on the **Configure group size and scaling policies** page, under **Instance scale\-in protection**, select the **Enable instance scale\-in protection** option\.

**To enable instance scale\-in protection \(AWS CLI\)**  
Use the following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to enable instance scale\-in protection\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg --new-instances-protected-from-scale-in ...
```

## Modify the instance scale\-in protection setting for a group<a name="instance-protection-modify"></a>

You can enable or disable the instance scale\-in protection setting for an Auto Scaling group\. When the instance scale\-in protection setting is enabled, all new instances launched after enabling it will have instance scale\-in protection enabled\. Previously launched instances are only protected from scale in if you enable the instance scale\-in protection setting for each instance individually\.

**To change the instance scale\-in protection setting for a group \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select check box next to the Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\.

1. For **Instance scale\-in protection**, select **Enable instance scale\-in protection**\.

1. Choose **Update**\.

**To change the instance scale\-in protection setting for a group \(AWS CLI\)**  
Use the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to enable instance scale\-in protection for the specified Auto Scaling group\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --new-instances-protected-from-scale-in
```

Use the following command to disable instance scale\-in protection for the specified group\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --no-new-instances-protected-from-scale-in
```

## Modify the instance scale\-in protection setting for an instance<a name="instance-protection-instance"></a>

By default, an instance gets its instance scale\-in protection setting from its Auto Scaling group\. However, you can enable or disable instance scale\-in protection for an instance at any time\.

**To change the instance scale\-in protection setting for an instance \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Instance management** tab, in **Instances**, select an instance\.

1. To enable instance scale\-in protection, choose **Actions**, **Set scale\-in protection**\. When prompted, choose **Set scale\-in protection**\.

1. To disable instance scale\-in protection, choose **Actions**, **Remove scale\-in protection**\. When prompted, choose **Remove scale\-in protection**\.

**To change the instance scale\-in protection setting for an instance \(AWS CLI\)**  
Use the following [set\-instance\-protection](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-protection.html) command to enable instance scale\-in protection for the specified instance\.

```
aws autoscaling set-instance-protection --instance-ids i-5f2e8a0d --auto-scaling-group-name my-asg --protected-from-scale-in
```

Use the following command to disable instance scale\-in protection for the specified instance\.

```
aws autoscaling set-instance-protection --instance-ids i-5f2e8a0d --auto-scaling-group-name my-asg --no-protected-from-scale-in
```