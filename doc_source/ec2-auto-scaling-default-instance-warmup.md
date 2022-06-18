# Set the default instance warmup for an Auto Scaling group<a name="ec2-auto-scaling-default-instance-warmup"></a>

You can use the default instance warmup feature to improve the Amazon CloudWatch metrics used for dynamic scaling\. CloudWatch collects and aggregates usage data, such as CPU and network I/O, across your Auto Scaling instances\. You use these metrics to create scaling policies that adjust the number of instances in your Auto Scaling group as the selected metric's value increases and decreases\.

When default instance warmup is not enabled, each instance starts contributing usage data to the aggregated metrics as soon as the instance reaches the `InService` state\. However, if you enable default instance warmup, this lets your instances finish warming up before they contribute the usage data\. This keeps dynamic scaling from being affected by metrics for individual instances that aren't handling application traffic yet and that might be experiencing temporarily high usage of compute resources\. 

Default instance warmup is not configured and is not enabled by default\. To optimize the performance of scaling policies that scale continuously, such as target tracking and step scaling policies, we strongly recommend that you enable the default instance warmup, *even if its value is set to 0 seconds*\. When default instance warmup is not enabled, the default cooldown might be applied when instances launch, and dynamic scaling scale\-in activities will be blocked until the default cooldown period ends\.

In addition, by enabling the default instance warmup feature, you no longer have to specify values for warm\-up parameters for individual scaling policies and instance refreshes\. You can use the default instance warmup to unify these settings at the group level\. By default, these parameters will be set to the value of the default instance warmup\. For more information, see [Available warm\-up and cooldown settings](consolidated-view-of-warm-up-and-cooldown-settings.md)\.

**Topics**
+ [Choose a time value for the default instance warmup](#determining-the-value-of-the-default-instance-warmup)
+ [Understand how the default instance warmup affects scaling](#understand-how-warmup-affects-scaling)
+ [Enable the default instance warmup for a group](#enable-default-instance-warmup)
+ [Verify the default instance warmup for a group](#verify-default-instance-warmup)

## Choose a time value for the default instance warmup<a name="determining-the-value-of-the-default-instance-warmup"></a>

When you choose a warm\-up time value, aim for an optimal balance between collecting usage data for legitimate traffic, but minimizing data collection associated with temporary usage spikes on startup\.

When instances finish launching, they're attached to the Auto Scaling group and registered to an Elastic Load Balancing load balancer, if used, before they enter the `InService` state\. After the instances enter the `InService` state, resource consumption can still experience temporary spikes and need time to stabilize\. For example, resource consumption for an application server that must download and cache large assets takes longer to stabilize than a lightweight web server with no large assets to download\. The default instance warmup provides the time delay necessary for resource consumption to stabilize\. 

**Important**  
You should also consider using lifecycle hooks for use cases where you have configuration tasks or scripts to run on startup\. Lifecycle hooks can delay instances from being put in service during a scale\-out event until they have finished initializing\. They are particularly useful if you have bootstrapping scripts that take a while to complete\. Therefore, if you add a lifecycle hook, you can reduce the value of the default instance warmup\. If you set the default instance warmup to 0 seconds, no additional waiting period is imposed beyond the time needed for the lifecycle hook\. For more information about using lifecycle hooks, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\. 

## Understand how the default instance warmup affects scaling<a name="understand-how-warmup-affects-scaling"></a>

After you enable the default instance warmup, each newly launched instance in your Auto Scaling group has a warm\-up period\. 

While instances are in the warm\-up period, your scaling policies only scale out if the metric value from instances that are not warming up is greater than the policy's alarm high threshold \(which is the same as the target utilization of a target tracking scaling policy\)\.

If instances are still warming up and the group scales out again, the instances are counted as part of the desired capacity for the next scale\-out activity\. Therefore, multiple alarm breaches that fall in the range of the same step adjustment result in a single scaling activity\. The intention is to continuously \(but not excessively\) scale out\.

If demand decreases, the intention is to scale in conservatively to protect your application's availability\. This blocks scale\-in activities initiated by scaling policies until the instances finish warming up\.

## Enable the default instance warmup for a group<a name="enable-default-instance-warmup"></a>

You can enable the default instance warmup when you create an Auto Scaling group\. You can also enable it for existing groups\. 

------
#### [ Console ]

**To enable the default instance warmup for a new group \(console\)**  
When you create the Auto Scaling group, on the **Configure advanced options** page, under **Additional settings**, select the **Enable default instance warmup** option\. Choose the warm\-up time that you need for your application\.

------
#### [ AWS CLI ]

**To enable the default instance warmup for a new group \(AWS CLI\)**  
To enable the default instance warmup for an Auto Scaling group, add the `--default-instance-warmup` option and specify a value, in seconds, from 0 to 3600\. After it's enabled, a value of `-1` will turn this setting off\.

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group with the name *my\-asg* and enables the default instance warmup with a value of *120* seconds\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg --default-instance-warmup 120 ...
```

**Tip**  
If this command throws an error, make sure that you have updated the AWS CLI locally to the latest version\.

------

------
#### [ Console ]

**To enable the default instance warmup for an existing group \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling](https://console.aws.amazon.com/ec2autoscaling) and choose the AWS Region that you created your Auto Scaling group in\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\.

1. For **Default instance warmup**, choose the warm\-up time that you need for your application\.

1. Choose **Update**\.

------
#### [ AWS CLI ]

**To enable the default instance warmup for an existing group \(AWS CLI\)**  
The following example uses the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to enable the default instance warmup with a value of *120* seconds for an existing Auto Scaling group named *my\-asg*\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --default-instance-warmup 120
```

**Tip**  
If this command throws an error, make sure that you have updated the AWS CLI locally to the latest version\.

------

## Verify the default instance warmup for a group<a name="verify-default-instance-warmup"></a>

**To verify the default instance warmup for an Auto Scaling group \(AWS CLI\)**  
Use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
```

The following is an example response\.

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupName": "my-asg",
            "AutoScalingGroupARN": "arn",
            ...
            "DefaultInstanceWarmup": 120
        }
    ]
}
```