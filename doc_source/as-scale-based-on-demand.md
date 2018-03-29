# Dynamic Scaling<a name="as-scale-based-on-demand"></a>

When you configure dynamic scaling, you must define how you want to scale in response to changing demand\. For example, say you have a web application that currently runs on two instances and you do not want the CPU utilization of the Auto Scaling group to exceed 70 percent\. You can configure your Auto Scaling group to scale automatically to meet this need\. The policy type determines how the scaling action is performed\.

**Topics**
+ [Scaling Policy Types](#as-scaling-types)
+ [Target Tracking Scaling Policies](as-scaling-target-tracking.md)
+ [Simple and Step Scaling Policies](as-scaling-simple-step.md)
+ [Add a Scaling Policy to an Existing Auto Scaling Group](policy-updating-console.md)
+ [Scaling Based on Amazon SQS](as-using-sqs-queue.md)

## Scaling Policy Types<a name="as-scaling-types"></a>

Amazon EC2 Auto Scaling supports the following types of scaling policies:
+ **Target tracking scaling**—Increase or decrease  the current capacity of the group based on a target value for a specific metric\. This is similar to the way that your thermostat maintains the temperature of your home – you select a temperature and the thermostat does the rest\.
+ **Step scaling**—Increase or decrease the current capacity of the group based on a set of scaling adjustments, known as *step adjustments*, that vary based on the size of the alarm breach\.
+ **Simple scaling**—Increase or decrease the current capacity of the group based on a single scaling adjustment\.

If you are scaling based on a utilization metric that increases or decreases proportionally to the number of instances in an Auto Scaling group, we recommend that you use target tracking scaling policies\. Otherwise, we recommend that you use step scaling policies\.