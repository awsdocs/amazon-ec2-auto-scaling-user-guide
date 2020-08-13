# Scaling the size of your Auto Scaling group<a name="scaling_plan"></a>

*Scaling* is the ability to increase or decrease the compute capacity of your application\. Scaling starts with an event, or scaling action, which instructs an Auto Scaling group to either launch or terminate Amazon EC2 instances\.

Amazon EC2 Auto Scaling provides a number of ways to adjust scaling to best meet the needs of your applications\. As a result, it's important that you have a good understanding of your application\. Keep the following considerations in mind:
+ What role should Amazon EC2 Auto Scaling play in your application's architecture? It's common to think about automatic scaling primarily as a way to increase and decrease capacity, but it's also useful for maintaining a steady number of servers\.
+ What cost constraints are important to you? Because Amazon EC2 Auto Scaling uses EC2 instances, you only pay for the resources that you use\. Knowing your cost constraints helps you decide when to scale your applications, and by how much\.
+ What metrics are important to your application? Amazon CloudWatch supports a number of different metrics that you can use with your Auto Scaling group\. 

**Topics**
+ [Scaling options](#scaling_typesof)
+ [Setting capacity limits](asg-capacity-limits.md)
+ [Maintaining a fixed number of instances](as-maintain-instance-levels.md)
+ [Manual scaling](as-manual-scaling.md)
+ [Dynamic scaling](as-scale-based-on-demand.md)
+ [Scaling cooldowns](Cooldown.md)
+ [Scheduled scaling](schedule_time.md)
+ [Auto Scaling instance termination](as-instance-termination.md)
+ [Lifecycle hooks](lifecycle-hooks.md)
+ [Temporarily removing instances](as-enter-exit-standby.md)
+ [Suspending scaling](as-suspend-resume-processes.md)

## Scaling options<a name="scaling_typesof"></a>

Amazon EC2 Auto Scaling provides several ways for you to scale your Auto Scaling group\.

**Maintain current instance levels at all times**  
You can configure your Auto Scaling group to maintain a specified number of running instances at all times\. To maintain the current instance levels, Amazon EC2 Auto Scaling performs a periodic health check on running instances within an Auto Scaling group\. When Amazon EC2 Auto Scaling finds an unhealthy instance, it terminates that instance and launches a new one\. For more information, see [Maintaining a fixed number of instances in your Auto Scaling group](as-maintain-instance-levels.md)\.

**Scale manually**  
Manual scaling is the most basic way to scale your resources, where you specify only the change in the maximum, minimum, or desired capacity of your Auto Scaling group\. Amazon EC2 Auto Scaling manages the process of creating or terminating instances to maintain the updated capacity\. For more information, see [Manual scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.

**Scale based on a schedule**  
Scaling by schedule means that scaling actions are performed automatically as a function of time and date\. This is useful when you know exactly when to increase or decrease the number of instances in your group, simply because the need arises on a predictable schedule\. For more information, see [Scheduled scaling for Amazon EC2 Auto Scaling](schedule_time.md)\.

**Scale based on demand**  
A more advanced way to scale your resources, using scaling policies, lets you define parameters that control the scaling process\. For example, let's say that you have a web application that currently runs on two instances and you want the CPU utilization of the Auto Scaling group to stay at around 50 percent when the load on the application changes\. This method is useful for scaling in response to changing conditions, when you don't know when those conditions will change\. You can set up Amazon EC2 Auto Scaling to respond for you\. For more information, see [Dynamic scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)\.

**Use predictive scaling**  
You can also use Amazon EC2 Auto Scaling in combination with AWS Auto Scaling to scale resources across multiple services\. AWS Auto Scaling can help you maintain optimal availability and performance by combining predictive scaling and dynamic scaling \(proactive and reactive approaches, respectively\) to scale your Amazon EC2 capacity faster\. For more information, see the [AWS Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/plans/userguide/)\.