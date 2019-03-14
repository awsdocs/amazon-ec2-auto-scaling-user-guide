# Dynamic Scaling for Amazon EC2 Auto Scaling<a name="as-scale-based-on-demand"></a>

When you configure dynamic scaling, you must define how to scale in response to changing demand\. For example, you have a web application that currently runs on two instances and you want the CPU utilization of the Auto Scaling group to stay at around 50 percent when the load on the application changes\. This gives you extra capacity to handle traffic spikes without maintaining an excessive amount of idle resources\. You can configure your Auto Scaling group to scale automatically to meet this need\. The policy type determines how the scaling action is performed\. 

**Topics**
+ [Scaling Policy Types](#as-scaling-types)
+ [Multiple Scaling Policies](#multiple-scaling-policy-resolution)
+ [Target Tracking Scaling Policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)
+ [Simple and Step Scaling Policies for Amazon EC2 Auto Scaling](as-scaling-simple-step.md)
+ [Add a Scaling Policy to an Existing Auto Scaling Group](policy-updating-console.md)
+ [Scaling Based on Amazon SQS](as-using-sqs-queue.md)

## Scaling Policy Types<a name="as-scaling-types"></a>

Amazon EC2 Auto Scaling supports the following types of scaling policies:
+ **Target tracking scaling**—Increase or decrease the current capacity of the group based on a target value for a specific metric\. This is similar to the way that your thermostat maintains the temperature of your home – you select a temperature and the thermostat does the rest\.
+ **Step scaling**—Increase or decrease the current capacity of the group based on a set of scaling adjustments, known as *step adjustments*, that vary based on the size of the alarm breach\.
+ **Simple scaling**—Increase or decrease the current capacity of the group based on a single scaling adjustment\.

If you are scaling based on a utilization metric that increases or decreases proportionally to the number of instances in an Auto Scaling group, we recommend that you use target tracking scaling policies\. Otherwise, we recommend that you use step scaling policies\. 

## Multiple Scaling Policies<a name="multiple-scaling-policy-resolution"></a>

In most cases, a target tracking scaling policy is sufficient to configure your Auto Scaling group to scale out or scale in automatically\. A target tracking scaling policy allows you to select a desired outcome and have the Auto Scaling group add and remove instances as needed to achieve that outcome\. 

For an advanced scaling configuration, your Auto Scaling group can have more than one scaling policy\. For example, you can define one or more target tracking scaling policies, one or more step scaling policies, or both\. This provides greater flexibility to cover multiple scenarios\. 

To illustrate how multiple policies work together, consider an application that uses an Auto Scaling group and an Amazon SQS queue to send requests to a single EC2 instance\. To help ensure that the application performs at optimum levels, there are two policies that control when the Auto Scaling group should scale out\. One is a target tracking policy that uses a custom metric to add and remove capacity based on the number of SQS messages in the queue\. The other is a step policy that uses the Amazon CloudWatch `CPUUtilization` metric to add capacity when the instance exceeds 90 percent utilization for a specified length of time\. 

When there are multiple policies in force at the same time, there's a chance that each policy could instruct the Auto Scaling group to scale out \(or in\) at the same time\. For example, it's possible that the EC2 instance could trigger the CloudWatch alarm for the `CPUUtilization` metric at the same time that the SQS queue triggers the alarm for the custom metric\. 

When these situations occur, Amazon EC2 Auto Scaling chooses the policy that provides the largest capacity for both scale out and scale in\. Suppose, for example, that the policy for CPU utilization launches one instance, while the policy for the SQS queue launches two instances\. If the scale\-out criteria for both policies are met at the same time, Amazon EC2 Auto Scaling gives precedence to the SQS queue policy\. This results in the Auto Scaling group launching two instances\. 

The approach of giving precedence to the policy that provides the largest capacity applies even when the policies use different criteria for scaling out\. For example, if one policy terminates three instances, another policy decreases capacity by 25 percent, and the group currently has eight instances, Amazon EC2 Auto Scaling gives precedence to the policy that provides the largest number of instances at that time\. This results in the Auto Scaling group terminating two instances \(25% of 8 = 2\)\.