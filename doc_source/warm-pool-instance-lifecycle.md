# Warm pool instance lifecycle<a name="warm-pool-instance-lifecycle"></a>

Instances in the warm pool maintain their own independent lifecycle to help you create the appropriate lifecycle actions for each transition\. This lifecycle is designed to facilitate the use of Event Notifications to invoke actions in a target service \(for example, a Lambda function\)\. 

Note that the APIs that you use to create and manage lifecycle hooks have not changed \(only the instance lifecycle has changed\)\.

An Amazon EC2 instance transitions through different states from the moment it launches through to its termination\. You can create lifecycle hooks to act on these event states, when an instance has transitioned from one state to another one\.

The following diagram shows the transition between each state:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/warm-pools-lifecycle-diagram.png)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

¹ If you opted to put warmed instances into a `Running` state, instances in the warmed pool transition from `Warmed:Pending` to `Warmed:Running` instead\.

As shown in the preceding diagram:
+ A lifecycle hook runs immediately after an instance is launched into the warm pool, when it transitions to the `Warmed:Pending:Wait` state\. 
+ Another lifecycle hook runs immediately before an instance enters the `InService` state, when it transitions to the `Pending:Wait` state\.
+ A final lifecycle hook runs immediately before the instance is terminated, when it transitions to the `Terminating:Wait` state\.

When you add lifecycle hooks, consider the following:
+ When you add a launch lifecycle hook, Amazon EC2 Auto Scaling pauses an instance that transitions to the `Warmed:Pending:Wait` state and the `Pending:Wait` state\.
+ If the demand on your application depletes the warm pool, Amazon EC2 Auto Scaling can launch instances directly into the Auto Scaling group if the group has not yet reached its maximum capacity\. In this case, your launch lifecycle hook pauses the instance only at the `Pending:Wait` state\.

These are some of the reasons that you might create lifecycle actions for instances leaving the warm pool:
+ You can use this time to prepare EC2 instances for use, for example, if you have services that must start for your application to work correctly\.
+ You want to pre\-populate cache data to ensure that a new server doesn't launch with a completely empty cache\.
+ You want to register new instances as managed instances with your configuration management service\.

## Supported notification targets<a name="warm-pools-supported-notification-targets"></a>

Amazon EC2 Auto Scaling can send lifecycle event notification messages to the following notification targets\.
+ Lambda functions \(via EventBridge\)
+ Amazon SNS topics 
+ Amazon SQS queues 

The following information can help you configure these notification targets when creating lifecycle hooks based on the following procedures:
+ [Route notifications to Lambda using EventBridge](configuring-lifecycle-hook-notifications.md#cloudwatch-events-notification)
+ [Receive notifications using Amazon SNS](configuring-lifecycle-hook-notifications.md#sns-notifications)
+ [Receive notifications using Amazon SQS](configuring-lifecycle-hook-notifications.md#sqs-notifications)

**EventBridge rules**: With Amazon EventBridge, you can create rules that trigger programmatic actions in response to warm pool events\. For example, you can create two EventBridge rules, one for when an instance is entering a warm pool, and one for when an instance is leaving the warm pool\. 

The EventBridge rules can trigger a Lambda function to perform programmatic actions on instances, depending on your use case and application\. The Lambda function is invoked with an incoming event emitted by the Auto Scaling group\. Lambda knows which Lambda function to invoke based on the event pattern in the EventBridge rule\. 

For more information, see [Creating EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md)\.

**Amazon SNS notification targets**: You can add lifecycle hooks to receive Amazon SNS notifications when a lifecycle action occurs\. These notifications are sent to recipients who are subscribed to the Amazon SNS topic that you specify\. 

Before you add a lifecycle hook that notifies an Amazon SNS topic, you must have already set up the topic and an IAM service role that gives Amazon EC2 Auto Scaling permission to publish to Amazon SNS\. 

To receive separate notifications about instances leaving and entering the warm pool, you must also have set up Amazon SNS filtering\.

**Amazon SQS notification targets**: You can add lifecycle hooks to an Auto Scaling group and use Amazon SQS to set up a notification target to receive notifications when a lifecycle action occurs\. These notifications are sent to the Amazon SQS queue that you specify\. 

Before you add a lifecycle hook that notifies an Amazon SQS queue, you must have already set up the queue and an IAM service role that gives Amazon EC2 Auto Scaling permission to send notification messages to Amazon SQS\. 

If you want the queue consumer to process separate notifications about instances leaving but not entering the warm pool \(or vice versa\), you must also have set up the queue consumer to parse the message and then act on the message if a specific attribute matches the desired value\.