# Dynamic scaling for Amazon EC2 Auto Scaling<a name="as-scale-based-on-demand"></a>

Dynamic scaling scales the capacity of your Auto Scaling group as traffic changes occur\.

Amazon EC2 Auto Scaling supports the following types of dynamic scaling policies:
+ **Target tracking scaling**—Increase and decrease the current capacity of the group based on a Amazon CloudWatch metric and a target value\. It works similar to the way that your thermostat maintains the temperature of your home—you select a temperature and the thermostat does the rest\.
+ **Step scaling**—Increase and decrease the current capacity of the group based on a set of scaling adjustments, known as *step adjustments*, that vary based on the size of the alarm breach\.
+ **Simple scaling**—Increase and decrease the current capacity of the group based on a single scaling adjustment, with a cooldown period between each scaling activity\.

If you are scaling based on a metric that increases or decreases proportionally to the number of instances in an Auto Scaling group, we recommend that you use target tracking scaling policies\. Otherwise, we recommend that you use step scaling policies\. 

With target tracking, an Auto Scaling group scales in direct proportion to the actual load on your application\. That means that in addition to meeting the immediate need for capacity in response to load changes, a target tracking policy can also adapt to load changes that take place over time, for example, due to seasonal variations\.

By default, new Auto Scaling groups start without any scaling policies\. When you use an Auto Scaling group without any form of dynamic scaling, it doesn't scale on it own unless you set up scheduled scaling or predictive scaling\.

**Topics**
+ [How dynamic scaling policies work](#as-how-scaling-policies-work)
+ [Multiple dynamic scaling policies](#multiple-scaling-policy-resolution)
+ [Target tracking scaling policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)
+ [Step and simple scaling policies for Amazon EC2 Auto Scaling](as-scaling-simple-step.md)
+ [Set default values for instance warmup or scaling cooldown](set-default-values-for-instance-warmup-or-scaling-cooldown.md)
+ [Scaling based on Amazon SQS](as-using-sqs-queue.md)
+ [Verify a scaling activity for an Auto Scaling group](as-verify-scaling-activity.md)
+ [Disable a scaling policy for an Auto Scaling group](as-enable-disable-scaling-policy.md)
+ [Delete a scaling policy](deleting-scaling-policy.md)
+ [Example scaling policies for the AWS Command Line Interface \(AWS CLI\)](examples-scaling-policies.md)

## How dynamic scaling policies work<a name="as-how-scaling-policies-work"></a>

A dynamic scaling policy instructs Amazon EC2 Auto Scaling to track a specific CloudWatch metric, and it defines what action to take when the associated CloudWatch alarm is in ALARM\. The metrics that are used to invoke the alarm state are an aggregation of metrics coming from all of the instances in the Auto Scaling group\. \(For example, let's say you have an Auto Scaling group with two instances where one instance is at 60 percent CPU and the other is at 40 percent CPU\. On average, they are at 50 percent CPU\.\) When the policy is in effect, Amazon EC2 Auto Scaling adjusts the group's desired capacity up or down when the threshold of an alarm is breached\.

When a dynamic scaling policy is invoked, if the capacity calculation produces a number outside of the minimum and maximum size range of the group, Amazon EC2 Auto Scaling ensures that the new capacity never goes outside of the minimum and maximum size limits\. Capacity is measured in one of two ways: using the same units that you chose when you set the desired capacity in terms of instances, or using capacity units \(if [instance weighting](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md) is applied\)\.
+ Example 1: An Auto Scaling group has a maximum capacity of 3, a current capacity of 2, and a dynamic scaling policy that adds 3 instances\. When invoking this policy, Amazon EC2 Auto Scaling adds only 1 instance to the group to prevent the group from exceeding its maximum size\. 
+ Example 2: An Auto Scaling group has a minimum capacity of 2, a current capacity of 3, and a dynamic scaling policy that removes 2 instances\. When invoking this policy, Amazon EC2 Auto Scaling removes only 1 instance from the group to prevent the group from becoming less than its minimum size\. 

When the desired capacity reaches the maximum size limit, scaling out stops\. If demand drops and capacity decreases, Amazon EC2 Auto Scaling can scale out again\. 

The exception is when you use instance weighting\. In this case, Amazon EC2 Auto Scaling can scale out above the maximum size limit, but only by up to your maximum instance weight\. Its intention is to get as close to the new desired capacity as possible but still adhere to the allocation strategies that are specified for the group\. The allocation strategies determine which instance types to launch\. The weights determine how many capacity units each instance contributes to the desired capacity of the group based on its instance type\.
+ Example 3: An Auto Scaling group has a maximum capacity of 12, a current capacity of 10, and a dynamic scaling policy that adds 5 capacity units\. Instance types have one of three weights assigned: 1, 4, or 6\. When invoking the policy, Amazon EC2 Auto Scaling chooses to launch an instance type with a weight of 6 based on the allocation strategy\. The result of this scale\-out event is a group with a desired capacity of 12 and a current capacity of 16\.

## Multiple dynamic scaling policies<a name="multiple-scaling-policy-resolution"></a>

In most cases, a target tracking scaling policy is sufficient to configure your Auto Scaling group to scale out and scale in automatically\. A target tracking scaling policy allows you to select a desired outcome and have the Auto Scaling group add and remove instances as needed to achieve that outcome\. 

For an advanced scaling configuration, your Auto Scaling group can have more than one scaling policy\. For example, you can define one or more target tracking scaling policies, one or more step scaling policies, or both\. This provides greater flexibility to cover multiple scenarios\. 

To illustrate how multiple dynamic scaling policies work together, consider an application that uses an Auto Scaling group and an Amazon SQS queue to send requests to a single EC2 instance\. To help ensure that the application performs at optimum levels, there are two policies that control when the Auto Scaling group should scale out\. One is a target tracking policy that uses a custom metric to add and remove capacity based on the number of SQS messages in the queue\. The other is a step scaling policy that uses the Amazon CloudWatch `CPUUtilization` metric to add capacity when the instance exceeds 90 percent utilization for a specified length of time\. 

When there are multiple policies in force at the same time, there's a chance that each policy could instruct the Auto Scaling group to scale out \(or in\) at the same time\. For example, it's possible that the `CPUUtilization` metric spikes and breaches the threshold of the CloudWatch alarm at the same time that the SQS custom metric spikes and breaches the threshold of the custom metric alarm\. 

When these situations occur, Amazon EC2 Auto Scaling chooses the policy that provides the largest capacity for both scale out and scale in\. Suppose, for example, that the policy for `CPUUtilization` launches one instance, while the policy for the SQS queue launches two instances\. If the scale\-out criteria for both policies are met at the same time, Amazon EC2 Auto Scaling gives precedence to the SQS queue policy\. This results in the Auto Scaling group launching two instances\. 

The approach of giving precedence to the policy that provides the largest capacity applies even when the policies use different criteria for scaling in\. For example, if one policy terminates three instances, another policy decreases the number of instances by 25 percent, and the group has eight instances at the time of scale in, Amazon EC2 Auto Scaling gives precedence to the policy that provides the largest number of instances for the group\. This results in the Auto Scaling group terminating two instances \(25 percent of 8 = 2\)\. The intention is to prevent Amazon EC2 Auto Scaling from removing too many instances\.

We recommend caution, however, when using target tracking scaling policies with step scaling policies because conflicts between these policies can cause undesirable behavior\. For example, if the step scaling policy initiates a scale\-in activity before the target tracking policy is ready to scale out, the scale\-in activity will not be blocked\. After the scale\-in activity completes, the target tracking policy could instruct the group to scale out again\. 
