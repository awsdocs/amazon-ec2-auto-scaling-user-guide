# Use lifecycle hooks with a warm pool<a name="warm-pool-instance-lifecycle"></a>

Instances in a warm pool maintain their own independent lifecycle to help you create the appropriate custom action for each transition\. This lifecycle is designed to help you to invoke actions in a target service \(for example, a Lambda function\) while an instance is still initializing and before it is put in service\. 

**Note**  
The API operations that you use to add and manage lifecycle hooks and complete lifecycle actions are not changed\. Only the instance lifecycle is changed\. 

For more information about adding a lifecycle hook, see [Add lifecycle hooks](adding-lifecycle-hooks.md)\. For more information about completing a lifecycle action, see [Complete a lifecycle action](completing-lifecycle-hooks.md)\.

For instances entering the warm pool, you might need a lifecycle hook for one of the following reasons:
+ You want to launch EC2 instances from an AMI that takes a long time to finish initializing\.
+ You want to run user data scripts to bootstrap the EC2 instances\.

For instances leaving the warm pool, you might need a lifecycle hook for one of the following reasons:
+ You can use some extra time to prepare EC2 instances for use\. For example, you might have services that must start when an instance restarts before your application can work correctly\.
+ You want to pre\-populate cache data so that a new server doesn't launch with an empty cache\.
+ You want to register new instances as managed instances with your configuration management service\.

## Lifecycle state transitions for instances in a warm pool<a name="lifecycle-state-transitions"></a>

An Auto Scaling instance can transition through many states as part of its lifecycle\.

The following diagram shows the transition between Auto Scaling states when you use a warm pool:

![\[The lifecycle state transitions for instances in a warm pool.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/warm-pools-lifecycle-diagram.png)![\[The lifecycle state transitions for instances in a warm pool.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[The lifecycle state transitions for instances in a warm pool.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

¹ This state varies based on the warm pool's pool state setting\. If the pool state is set to `Running`, then this state is `Warmed:Running` instead\. If the pool state is set to `Hibernated`, then this state is `Warmed:Hibernated` instead\.

When you add lifecycle hooks, consider the following:
+ When a lifecycle hook is configured for the `autoscaling:EC2_INSTANCE_LAUNCHING` lifecycle action, a newly launched instance first pauses to perform a custom action when it reaches the `Warmed:Pending:Wait` state, and then again when the instance restarts and reaches the `Pending:Wait` state\.
+ When a lifecycle hook is configured for the `EC2_INSTANCE_TERMINATING` lifecycle action, a terminating instance pauses to perform a custom action when it reaches the `Terminating:Wait` state\. However, if you specify an instance reuse policy to return instances to the warm pool on scale in instead of terminating them, then an instance that is returning to the warm pool pauses to perform a custom action at the `Warmed:Pending:Wait` state for the `EC2_INSTANCE_TERMINATING` lifecycle action\.
+ If the demand on your application depletes the warm pool, Amazon EC2 Auto Scaling can launch instances directly into the Auto Scaling group as long as the group isn't at its maximum capacity yet\. If the instances launch directly into the group, they are only paused to perform a custom action at the `Pending:Wait` state\.
+ To control how long an instance stays in a wait state before it transitions to the next state, configure your custom action to use the complete\-lifecycle\-action command\. With lifecycle hooks, instances remain in a wait state either until you notify Amazon EC2 Auto Scaling that the specified lifecycle action is complete, or until the timeout period ends \(one hour by default\)\. 

The following summarizes the flow for a scale\-out event\.

![\[A flow diagram of a scale-out event.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/warm-pools-scale-out-event-diagram.png)![\[A flow diagram of a scale-out event.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[A flow diagram of a scale-out event.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

When instances reach a wait state, Amazon EC2 Auto Scaling sends a notification\. Examples of these notifications are available in the EventBridge section of this guide\. For more information, see [Warm pool event types and patterns](warm-pools-eventbridge-events.md)\.

## Supported notification targets<a name="warm-pools-supported-notification-targets"></a>

Amazon EC2 Auto Scaling provides support for defining any of the following as notification targets for lifecycle notifications:
+ EventBridge rules
+ Amazon SNS topics 
+ Amazon SQS queues

**Important**  
Remember, if you have a user data \(cloud\-init\) script in your launch template or launch configuration that configures your instances when they launch, you do not need to receive notifications to perform custom actions on instances that are starting or restarting\.

The following sections contain links to documentation that describes how to configure notification targets:

**EventBridge rules**: To run code when Amazon EC2 Auto Scaling puts an instance into a wait state, you can create an EventBridge rule and specify a Lambda function as its target\. To invoke different Lambda functions based on different lifecycle notifications, you can create multiple rules and associate each rule with a specific event pattern and Lambda function\. For more information, see [Create EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md)\.

**Amazon SNS topics**: To receive a notification when an instance is put into a wait state, you create an Amazon SNS topic and then set up Amazon SNS message filtering to deliver lifecycle notifications differently based on a message attribute\. For more information, see [Receive notifications using Amazon SNS](prepare-for-lifecycle-notifications.md#sns-notifications)\.

**Amazon SQS queues**: To set up a delivery point for lifecycle notifications where a relevant consumer can pick them up and process them, you can create an Amazon SQS queue and a queue consumer that processes messages from the SQS queue\. If you want the queue consumer to process lifecycle notifications differently based on a message attribute, you must also set up the queue consumer to parse the message and then act on the message when a specific attribute matches the desired value\. For more information, see [Receive notifications using Amazon SQS](prepare-for-lifecycle-notifications.md#sqs-notifications)\.