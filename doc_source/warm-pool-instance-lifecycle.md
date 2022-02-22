# Using lifecycle hooks with a warm pool<a name="warm-pool-instance-lifecycle"></a>

Instances in a warm pool maintain their own independent lifecycle to help you create the appropriate custom action for each transition\. This lifecycle is designed to help you to invoke actions in a target service \(for example, a Lambda function\) while an instance is still initializing and before it is put in service\. 

**Note**  
The API operations that you use to add and manage lifecycle hooks and complete lifecycle actions are not changed\. Only the instance lifecycle is changed\. 

For more information about adding a lifecycle hook, see [Adding lifecycle hooks](adding-lifecycle-hooks.md)\. For more information about completing a lifecycle action, see [Completing a lifecycle action](completing-lifecycle-hooks.md)\.

For instances entering the warm pool, you might need a lifecycle hook for one of the following reasons:
+ You want to launch EC2 instances from an AMI that takes a long time to finish initializing\.
+ You want to run user data scripts to bootstrap the EC2 instances\.

For instances leaving the warm pool, you might need a lifecycle hook for one of the following reasons:
+ You can use some extra time to prepare EC2 instances for use\. For example, you might have services that must start when you restart an instance before your application can work correctly\.
+ You want to pre\-populate cache data so that a new server doesn't launch with an empty cache\.
+ You want to register new instances as managed instances with your configuration management service\.

## Lifecycle states<a name="lifecycle-states"></a>

An Auto Scaling instance can transition through many states as part of its lifecycle\.

The following diagram shows the transition between Auto Scaling states when you use a warm pool:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/warm-pools-lifecycle-diagram.png)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

¹ This state varies based on the warm pool's pool state setting\. If the pool state is set to `Running`, then this state is `Warmed:Running` instead\.

When you add lifecycle hooks, consider the following:
+ When you create a warm pool, Amazon EC2 Auto Scaling launches and then puts one or more instances into a wait state \(`Warmed:Pending:Wait`\) before the instances transition into the `Stopped` or `Running` state\.
+ When your Auto Scaling group scales out, Amazon EC2 Auto Scaling puts one or more warm pool instances into a wait state \(`Pending:Wait`\) before the instances transition into the `InService` state\.
+ If the demand on your application depletes the warm pool, Amazon EC2 Auto Scaling can launch instances directly into the Auto Scaling group as long as the group isn't at its maximum capacity yet\. If the instances launch directly into the group, they are only put in the `Pending:Wait` state before they transition into the `InService` state\.

When instances reach a wait state, Amazon EC2 Auto Scaling sends a notification that includes the origin and the destination\. 

## Supported notification targets<a name="warm-pools-supported-notification-targets"></a>

Amazon EC2 Auto Scaling provides support for defining any of the following as notification targets for lifecycle notifications:
+ EventBridge rules
+ Amazon SNS topics 
+ Amazon SQS queues

The following sections contain links to documentation that describes how to configure notification targets:

**EventBridge rules**: To run code when Amazon EC2 Auto Scaling puts an instance into a wait state, you can create an EventBridge rule and specify a Lambda function as its target\. To invoke different Lambda functions based on different lifecycle notifications, you can create multiple rules and associate each rule with a specific event pattern and Lambda function\. For more information, see [Creating EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md)\.

**Amazon SNS topics**: To receive a notification when an instance is put into a wait state, you create an Amazon SNS topic and then set up Amazon SNS message filtering to deliver lifecycle notifications differently based on a message attribute\. For more information, see [Receive notifications using Amazon SNS](configuring-lifecycle-hook-notifications.md#sns-notifications)\.

**Amazon SQS queues**: To set up a delivery point for lifecycle notifications where a relevant consumer can pick them up and process them, you can create an Amazon SQS queue and a queue consumer that processes messages from the SQS queue\. If you want the queue consumer to process separate notifications based on a message attribute, you must also set up the queue consumer to parse the message and then act on the message when a specific attribute matches the desired value\. For more information, see [Receive notifications using Amazon SQS](configuring-lifecycle-hook-notifications.md#sqs-notifications)\.