# Amazon EC2 Auto Scaling lifecycle hooks<a name="lifecycle-hooks"></a>

Amazon EC2 Auto Scaling offers the ability to add lifecycle hooks to your Auto Scaling groups\. These hooks let you create solutions that are aware of events in the Auto Scaling instance lifecycle, and then perform a custom action on instances when the corresponding lifecycle event occurs\. A lifecycle hook provides a specified amount of time \(one hour by default\) to wait for the action to complete before the instance transitions to the next state\.

As an example of using lifecycle hooks with Auto Scaling instances: 
+ When a scale\-out event occurs, your newly launched instance completes its startup sequence and transitions to a wait state\. While the instance is in a wait state, it runs a script to download and install the needed software packages for your application, making sure that your instance is fully ready before it starts receiving traffic\. When the script is finished installing software, it sends the complete\-lifecycle\-action command to continue\.
+ When a scale\-in event occurs, a lifecycle hook pauses the instance before it is terminated and sends you a notification using Amazon EventBridge\. While the instance is in the wait state, you can invoke an AWS Lambda function or connect to the instance to download logs or other data before the instance is fully terminated\. 

A popular use of lifecycle hooks is to control when instances are registered with Elastic Load Balancing\. By adding a launch lifecycle hook to your Auto Scaling group, you can ensure that your bootstrap scripts have completed successfully and the applications on the instances are ready to accept traffic before they are registered to the load balancer at the end of the lifecycle hook\.

For an introduction video, see [AWS re:Invent 2018: Capacity Management Made Easy with Amazon EC2 Auto Scaling](https://youtu.be/PideBMIcwBQ?t=469) on *YouTube*\.

**Topics**
+ [Considerations and limitations for lifecycle hooks](#lifecycle-hook-considerations)
+ [Lifecycle hook availability](#lifecycle-hooks-availability)
+ [Examples](#lifecycle-hook-examples)
+ [How lifecycle hooks work](lifecycle-hooks-overview.md)
+ [Prepare to add a lifecycle hook](prepare-for-lifecycle-notifications.md)
+ [Add lifecycle hooks](adding-lifecycle-hooks.md)
+ [Complete a lifecycle action](completing-lifecycle-hooks.md)
+ [Tutorial: Configure user data to retrieve the target lifecycle state through instance metadata](tutorial-lifecycle-hook-instance-metadata.md)
+ [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)

## Considerations and limitations for lifecycle hooks<a name="lifecycle-hook-considerations"></a>

When using lifecycle hooks, keep in mind the following considerations and limitations:
+ Amazon EC2 Auto Scaling provides its own lifecycle to help with the management of Auto Scaling groups\. This lifecycle differs from that of other EC2 instances\. For more information, see [Amazon EC2 Auto Scaling instance lifecycle](ec2-auto-scaling-lifecycle.md)\.
+ Instances in a warm pool also have their own lifecycle, as described in [Lifecycle state transitions for instances in a warm pool](warm-pool-instance-lifecycle.md#lifecycle-state-transitions)\.
+ You can use lifecycle hooks with Spot Instances, but a lifecycle hook does not prevent an instance from terminating in the event that capacity is no longer available, which can happen at any time with a two\-minute interruption notice\. For more information, see [Spot Instance interruptions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html) in the *Amazon EC2 User Guide for Linux Instances*\. However, you can enable Capacity Rebalancing to proactively replace Spot Instances that have received a rebalance recommendation from the Amazon EC2 Spot service, a signal that is sent when a Spot Instance is at elevated risk of interruption\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot Interruptions](ec2-auto-scaling-capacity-rebalancing.md)\.
+ When scaling out, Amazon EC2 Auto Scaling doesn't count a new instance towards the aggregated CloudWatch instance metrics of the Auto Scaling group \(such as CPUUtilization, NetworkIn, NetworkOut, and so on\) until after the launch lifecycle hook finishes\. When default instance warmup is not enabled, or enabled but set to 0, Auto Scaling instances start contributing usage data to the aggregated instance metrics as soon as they reach the `InService` state\. For more information, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.

  On scale in, the aggregated instance metrics might not instantly reflect the removal of a terminating instance\. The terminating instance stops counting toward the group's aggregated instance metrics shortly after the Amazon EC2 Auto Scaling termination workflow begins\. 
+ When an Auto Scaling group launches or terminates instances, scaling activities initiated by simple scaling policies are paused\. If lifecycle hooks are invoked, scaling activities due to simple scaling policies are paused until the lifecycle actions have completed and the cooldown period has expired\. Setting a long interval for the cooldown period means that it will take longer for scaling to resume\. For more information, see [Scaling cooldowns for Amazon EC2 Auto Scaling](ec2-auto-scaling-scaling-cooldowns.md)\.
+ Instances can remain in a wait state for a finite period of time\. The default timeout for a lifecycle hook is one hour \(heartbeat timeout\)\. There is also a global timeout that specifies the maximum amount of time that you can keep an instance in a wait state\. The global timeout is 48 hours or 100 times the heartbeat timeout, whichever is smaller\.

When creating lifecycle hooks, keep in mind the following points:
+ You can configure a launch lifecycle hook to abandon the launch if an unexpected failure occurs, in which case Amazon EC2 Auto Scaling automatically terminates and replaces the instance\.
+ Amazon EC2 Auto Scaling limits the rate at which it allows instances to launch if the lifecycle hooks are failing consistently, so make sure to test and fix any permanent errors in your lifecycle actions\. 
+ Creating and updating lifecycle hooks using the AWS CLI, AWS CloudFormation, or an SDK provides options not available when creating a lifecycle hook from the AWS Management Console\. For example, the field to specify the ARN of an SNS topic or SQS queue doesn't appear in the console, because Amazon EC2 Auto Scaling already sends events to Amazon EventBridge\. These events can be filtered and redirected to AWS services such as Lambda, Amazon SNS, and Amazon SQS as needed\.
+ You can add multiple lifecycle hooks to an Auto Scaling group while you are creating it, by calling the [CreateAutoScalingGroup](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateAutoScalingGroup.html) API using the AWS CLI, AWS CloudFormation, or an SDK\. However, each hook must have the same notification target and IAM role, if specified\. To create lifecycle hooks with different notification targets and different roles, create the lifecycle hooks one at a time in separate calls to the [PutLifecycleHook](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutLifecycleHook.html) API\. 

## Lifecycle hook availability<a name="lifecycle-hooks-availability"></a>

The following table lists the lifecycle hooks available for various scenarios\.


| Event | Instance launch or termination¹ | [Maximum Instance Lifetime](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-max-instance-lifetime.html): Replacement instances | [Instance Refresh](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html): Replacement instances | [Capacity Rebalancing](https://docs.aws.amazon.com/autoscaling/ec2/userguide/capacity-rebalance.html): Replacement instances | [Warm Pools](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-warm-pools.html): Instances entering and leaving the warm pool | 
| --- | --- | --- | --- | --- | --- | 
| Instance launching | ✓ | ✓ | ✓ | ✓ | ✓ | 
| Instance terminating | ✓ | ✓ | ✓ | ✓ | ✓ | 

¹ Applies to instances launched or terminated when the group is created or deleted, when the group scales automatically, or when you manually adjust your group's desired capacity\. Does not apply when you attach or detach instances, move instances in and out of standby mode, or delete the group with the force delete option\.

## Examples<a name="lifecycle-hook-examples"></a>

We provide a few JSON and YAML template snippets that you can use to understand how to declare lifecycle hooks in your AWS CloudFormation stack templates\. For more information, see the [AWS::AutoScaling::LifecycleHook](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-autoscaling-lifecyclehook.html) reference in the *AWS CloudFormation User Guide*\.

You can also visit our [GitHub repository](https://github.com/aws-samples/amazon-ec2-auto-scaling-group-examples) to download example templates and user data scripts for lifecycle hooks\.

For other examples of the use of lifecycle hooks, see the following blog posts: [Building a Backup System for Scaled Instances using Lambda and Amazon EC2 Run Command](http://aws.amazon.com/blogs/compute/building-a-backup-system-for-scaled-instances-using-aws-lambda-and-amazon-ec2-run-command) and [Run code before terminating an EC2 Auto Scaling instance](http://aws.amazon.com/blogs/infrastructure-and-automation/run-code-before-terminating-an-ec2-auto-scaling-instance)\.