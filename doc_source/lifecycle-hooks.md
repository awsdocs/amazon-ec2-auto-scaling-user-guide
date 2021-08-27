# Amazon EC2 Auto Scaling lifecycle hooks<a name="lifecycle-hooks"></a>

Amazon EC2 Auto Scaling offers the ability to add lifecycle hooks to your Auto Scaling groups\. These hooks enable an Auto Scaling group to be aware of events in the Auto Scaling instance lifecycle, and then perform a custom action when the corresponding lifecycle event occurs\. A lifecycle hook provides a specified amount of time \(one hour by default\) to complete the lifecycle action before the instance transitions to the next state\. 

As an example of using lifecycle hooks with Auto Scaling instances: 
+ When a scale\-out event occurs, your newly launched instance completes its startup sequence and transitions to a wait state\. While the instance is in a wait state, it runs a script to download and install the needed software packages for your application, making sure that your instance is fully ready before it starts receiving traffic\. When the script is finished installing software, it sends the complete\-lifecycle\-action command to continue\.
+ When a scale\-in event occurs, a lifecycle hook pauses the instance before it is terminated and sends you a notification using Amazon EventBridge\. While the instance is in the wait state, you can invoke an AWS Lambda function or connect to the instance to download logs or other data before the instance is fully terminated\. 

A popular use of lifecycle hooks is to control when instances are registered with Elastic Load Balancing\. By adding a launch lifecycle hook to your Auto Scaling group, you can ensure that the applications on the instances are ready to accept traffic before they are registered to the load balancer at the end of the lifecycle hook\. 

**Topics**
+ [How lifecycle hooks work](#lifecycle-hooks-overview)
+ [Considerations and limitations](#lifecycle-hook-considerations)
+ [Receiving lifecycle hook notifications](#preparing-for-notification)
+ [GitHub repository](#github-repository-lifecycle-hook-templates)
+ [Lifecycle hook availability](#lifecycle-hooks-availability)
+ [Configuring a notification target for a lifecycle hook](configuring-lifecycle-hook-notifications.md)
+ [Adding lifecycle hooks](adding-lifecycle-hooks.md)
+ [Completing a lifecycle action](completing-lifecycle-hooks.md)
+ [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)

**Note**  
Amazon EC2 Auto Scaling provides its own lifecycle to help with the deployment of Auto Scaling groups\. This lifecycle differs from that of other EC2 instances\. For more information, see [Amazon EC2 Auto Scaling instance lifecycle](AutoScalingGroupLifecycle.md)\.

## How lifecycle hooks work<a name="lifecycle-hooks-overview"></a>

After you add lifecycle hooks to your Auto Scaling group, they work as follows:

1. The Auto Scaling group responds to a scale\-out event and begins launching an instance\.

1. The lifecycle hook puts the instance into a wait state \(`Pending:Wait`\) and then performs a custom action\.

   The instance remains in a wait state either until you complete the lifecycle action using the [complete\-lifecycle\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) CLI command or the [CompleteLifecycleAction](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CompleteLifecycleAction.html) action, or until the timeout period ends\. By default, the instance remains in a wait state for one hour, and then the Auto Scaling group continues the launch process \(`Pending:Proceed`\)\. If you need more time, you can restart the timeout period by recording a heartbeat\. If you complete the lifecycle action before the timeout period expires, the period ends and the Auto Scaling group continues the launch process\.

1. The instance enters the `InService` state and the health check grace period starts\. If the Auto Scaling group is associated with an Elastic Load Balancing load balancer, the instance is registered with the load balancer, and the load balancer starts checking its health\. After the health check grace period ends, Amazon EC2 Auto Scaling begins checking the health state of the instance\.

1. The Auto Scaling group responds to a scale\-in event and begins terminating an instance\. If the Auto Scaling group is being used with Elastic Load Balancing, the terminating instance is first deregistered from the load balancer\. If connection draining is enabled for the load balancer, the instance stops accepting new connections and waits for existing connections to drain before completing the deregistration process\.

1. The lifecycle hook puts the instance into a wait state \(`Terminating:Wait`\) and then performs a custom action\.

   The instance remains in a wait state either until you complete the lifecycle action, or until the timeout period ends \(one hour by default\)\. After you complete the lifecycle hook or the timeout period expires, the instance transitions to the next state \(`Terminating:Proceed`\)\.

1. The instance is terminated\.

The following illustration shows the transitions between Auto Scaling instance states in this process\. 

![\[The lifecycle of instances using lifecycle hooks.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/lifecycle_hooks.png)

## Considerations and limitations<a name="lifecycle-hook-considerations"></a>

When using lifecycle hooks, keep in mind the following key considerations and limitations:
+ You can configure a launch lifecycle hook to abandon the launch if an unexpected failure occurs, in which case Amazon EC2 Auto Scaling automatically terminates and replaces the instance\.
+ Amazon EC2 Auto Scaling limits the rate at which it allows instances to launch if the lifecycle hooks are failing consistently\.
+ You can use lifecycle hooks with Spot Instances\. However, a lifecycle hook does not prevent an instance from terminating in the event that capacity is no longer available, which can happen at any time with a two\-minute interruption notice\. For more information, see [Spot Instance interruptions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html) in the *Amazon EC2 User Guide for Linux Instances*\. 
+ When scaling out, Amazon EC2 Auto Scaling doesn't count a new instance towards the aggregated CloudWatch metrics of the Auto Scaling group until after the launch lifecycle hook finishes \(and the scaling policy's warm\-up time completes\)\. On scale in, the terminating instance continues to count as part of the group's aggregated metrics until after the termination lifecycle hook finishes\.
+ When an Auto Scaling group launches or terminates an instance due to simple scaling policies, a cooldown period takes effect\. The cooldown period helps ensure that the Auto Scaling group does not launch or terminate more instances than needed before the effects of previous simple scaling activities are visible\. When a lifecycle action occurs, and an instance enters the wait state, scaling activities due to simple scaling policies are paused\. When the lifecycle actions finish, the cooldown period starts\. If you set a long interval for the cooldown period, it will take more time for scaling to resume\. For more information, see [Scaling cooldowns for Amazon EC2 Auto Scaling](Cooldown.md)\.
+ There is a quota that specifies the maximum number of lifecycle hooks that you can create per Auto Scaling group\. For more information, see [Amazon EC2 Auto Scaling service quotas](as-account-limits.md)\. 
+ For examples of the use of lifecycle hooks, see the following blog posts: [Building a Backup System for Scaled Instances using Lambda and Amazon EC2 Run Command](http://aws.amazon.com/blogs/compute/building-a-backup-system-for-scaled-instances-using-aws-lambda-and-amazon-ec2-run-command) and [Run code before terminating an EC2 Auto Scaling instance](http://aws.amazon.com/blogs/infrastructure-and-automation/run-code-before-terminating-an-ec2-auto-scaling-instance)\.

### Keeping instances in a wait state<a name="lifecycle-hook-wait-state"></a>

Instances can remain in a wait state for a finite period of time\. The default is one hour \(3600 seconds\)\. 

You can adjust how long the timeout period lasts in the following ways:
+ Set the heartbeat timeout for the lifecycle hook when you create the lifecycle hook\. With the [put\-lifecycle\-hook](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-lifecycle-hook.html) command, use the `--heartbeat-timeout` option\.
+ Continue to the next state if you finish before the timeout period ends, using the [complete\-lifecycle\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) command\.
+ Postpone the end of the timeout period by recording a heartbeat, using the [record\-lifecycle\-action\-heartbeat](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/record-lifecycle-action-heartbeat.html) command\. This extends the timeout period by the timeout value specified when you created the lifecycle hook\. For example, if the timeout value is one hour, and you call this command after 30 minutes, the instance remains in a wait state for an additional hour, or a total of 90 minutes\.

The maximum amount of time that you can keep an instance in a wait state is 48 hours or 100 times the heartbeat timeout, whichever is smaller\.

## Receiving lifecycle hook notifications<a name="preparing-for-notification"></a>

You can configure various EventBridge rules to invoke a Lambda function or send a notification when an instance enters a wait state and Amazon EC2 Auto Scaling submits an event for a lifecycle action to EventBridge\. The event contains information about the instance that is launching or terminating, and a token that you can use to control the lifecycle action\. For an introductory tutorial\-style guide for creating lifecycle hooks, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\. For examples of the events that are emitted when a lifecycle action occurs, see [Auto Scaling events](cloud-watch-events.md#cloudwatch-event-types)\.

Alternatively, if you have a user data or cloud\-init script that configures your instances after they launch, you do not need to configure a notification\. The script can control the lifecycle action using the ID of the instance on which it runs\. If you are not doing so already, update your script to retrieve the instance ID of the instance from the instance metadata\. For more information, see [Retrieving instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) in the *Amazon EC2 User Guide for Linux Instances*\. You can have the script signal the lifecycle hook when the configuration script is complete, to allow the instance to proceed to the next state\.

**Note**  
The Amazon EC2 Auto Scaling console does not provide the option to define an Amazon SNS or Amazon SQS notification target for the lifecycle hook\. These lifecycle hooks must be added using either the AWS CLI or one of the SDKs\. For more information, see [Configuring notifications](configuring-lifecycle-hook-notifications.md)\. 

## GitHub repository<a name="github-repository-lifecycle-hook-templates"></a>

You can visit our [GitHub repository](https://github.com/aws-samples/amazon-ec2-auto-scaling-group-examples) to download templates and scripts for lifecycle hooks\.

## Lifecycle hook availability<a name="lifecycle-hooks-availability"></a>

The following table lists the lifecycle hooks available for various scenarios\.


| Event | Instance launch or termination¹ | [Maximum Instance Lifetime](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-max-instance-lifetime.html): Replacement instances | [Instance Refresh](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html): Replacement instances | [Capacity Rebalancing](https://docs.aws.amazon.com/autoscaling/ec2/userguide/capacity-rebalance.html): Replacement instances | [Warm Pools](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-warm-pools.html): Instances entering and leaving the warm pool | 
| --- | --- | --- | --- | --- | --- | 
| Instance launching | ✓ | ✓ | ✓ | ✓ | ✓ | 
| Instance terminating | ✓ | ✓ | ✓ | ✓ | ✓ | 

¹ Applies to instances launched or terminated when the group is created or deleted, when the group scales automatically, or when you manually adjust your group's desired capacity\. Does not apply when you attach or detach instances, move instances in and out of standby mode, or delete the group with the force delete option\.