# Scaling cooldowns for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-scaling-cooldowns"></a>

After your Auto Scaling group launches or terminates instances, it waits for a cooldown period to end before any further scaling activities initiated by simple scaling policies can start\. The intention of the cooldown period is to prevent your Auto Scaling group from launching or terminating additional instances before the effects of previous activities are visible\. 

**Important**  
As a best practice, we recommend that you do not use simple scaling policies and scaling cooldowns\.   
In most cases, a target tracking scaling policy or a step scaling policy is better for scaling performance\. For a scaling policy that changes the size of your Auto Scaling group proportionally as the value of the scaling metric decreases or increases, we recommend [target tracking](as-scaling-target-tracking.md) over either simple scaling or step scaling\.

Suppose, for example, that a simple scaling policy for CPU utilization recommends launching two instances\. Amazon EC2 Auto Scaling launches two instances and then pauses the scaling activities until the cooldown period ends\. After the cooldown period ends, any scaling activities initiated by simple scaling policies can resume\. If CPU utilization breaches the alarm high threshold again, the Auto Scaling group scales out again, and the cooldown period takes effect again\. However, if two instances were enough to bring the metric value back down, the group remains at its current size\.

**Topics**
+ [Considerations](#cooldown-considerations)
+ [Lifecycle hooks cause additional delays](#cooldowns-lifecycle-hooks)
+ [Change the default cooldown period](#set-default-cooldown)
+ [Set a cooldown period for specific simple scaling policies](#cooldowns-scaling-specific)

## Considerations<a name="cooldown-considerations"></a>

The following considerations apply when working with simple scaling policies and scaling cooldowns:
+ Target tracking and step scaling policies can initiate a scale\-out activity immediately without waiting for the cooldown period to end\. Instead, whenever your Auto Scaling group launches instances, the individual instances have a warm\-up period\. For more information, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.
+ When a scheduled action starts at the scheduled time, it can also initiate a scaling activity immediately without waiting for the cooldown period to end\. 
+ If an instance becomes unhealthy, Amazon EC2 Auto Scaling does not wait for the cooldown period to end before replacing the unhealthy instance\.
+ When multiple instances launch or terminate, the cooldown period \(either the default cooldown or the scaling policy\-specific cooldown\) takes effect starting when the last instance finishes launching or terminating\.
+ When you manually scale your Auto Scaling group, the default is not to wait for a cooldown to end\. However, you can override this behavior and honor the default cooldown when you use the AWS CLI or an SDK to manually scale\. 
+ By default, Elastic Load Balancing waits 300 seconds to complete the deregistration \(connection draining\) process\. If the group is behind an Elastic Load Balancing load balancer, it will wait for the terminating instances to deregister before starting the cooldown period\.

## Lifecycle hooks cause additional delays<a name="cooldowns-lifecycle-hooks"></a>

If a [lifecycle hook](lifecycle-hooks.md) is invoked, the cooldown period begins after you complete the lifecycle action or after the timeout period ends\. For example, consider an Auto Scaling group that has a lifecycle hook for instance launch\. When the application experiences an increase in demand, the group launches an instance to add capacity\. Because there is a lifecycle hook, the instance is put into a wait state and scaling activities due to simple scaling policies are paused\. When the instance enters the `InService` state, the cooldown period starts\. When the cooldown period ends, simple scaling policy activities are resumed\.

When Elastic Load Balancing is enabled, for the purposes of scaling in, the cooldown period starts when the terminating instance finishes connection draining \(deregistration delay\)\. It doesn't wait for the hook duration\. This means that any scaling activities due to simple scaling policies can resume as soon as the result of the scale\-in event is reflected in the capacity of the group\. Otherwise, waiting to complete all three activities—connection draining, a lifecycle hook, and a cooldown period— would significantly increase the amount of time that the Auto Scaling group needs to pause scaling\.

## Change the default cooldown period<a name="set-default-cooldown"></a>

You can't set the default cooldown when you initially create an Auto Scaling group in the Amazon EC2 Auto Scaling console\. By default, this cooldown period is set to 300 seconds \(5 minutes\)\. If needed, you can update this after the group is created\. 

**To change the default cooldown period \(console\)**  
After creating the Auto Scaling group, on the **Details** tab, choose **Advanced configurations**, **Edit**\. For **Default cooldown**, choose the amount of time that you want based on your instance startup time or other application needs\.

**To change the default cooldown period \(AWS CLI\)**  
Use the following commands to change the default cooldown for new or existing Auto Scaling groups\. If the default cooldown is not defined, the default value of 300 seconds is used\. 
+ [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html)
+ [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html)

To confirm the value of the default cooldown, use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

## Set a cooldown period for specific simple scaling policies<a name="cooldowns-scaling-specific"></a>

By default, all simple scaling policies use the default cooldown period that is defined for the Auto Scaling group\. To set a cooldown period for specific simple scaling policies, use the optional cooldown parameter when you create or update the policy\. When a cooldown period is specified for a policy, it overrides the default cooldown\.

One common use for a scaling policy\-specific cooldown period is with a scale\-in policy\. Because this policy terminates instances, Amazon EC2 Auto Scaling needs less time to determine whether to terminate additional instances\. Terminating instances should be a much quicker operation than launching instances\. The default cooldown period of 300 seconds is therefore too long\. In this case, a scaling policy\-specific cooldown period with a lower value for your scale\-in policy can help you reduce costs by allowing the group to scale in faster\. 

To create or update simple scaling policies in the console, choose the **Automatic scaling** tab after you create the group\. To create or update simple scaling policies using the AWS CLI, use the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command\. For more information, see [Step and simple scaling policies](as-scaling-simple-step.md)\.