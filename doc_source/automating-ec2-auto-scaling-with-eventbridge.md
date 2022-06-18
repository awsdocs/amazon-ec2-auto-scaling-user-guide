# Use EventBridge to handle Auto Scaling events<a name="automating-ec2-auto-scaling-with-eventbridge"></a>

Amazon EventBridge, formerly called CloudWatch Events, helps you set up event\-driven rules that monitor resources and initiate target actions that use other AWS services\.

Events from Amazon EC2 Auto Scaling are delivered to EventBridge in near real time\. You can establish EventBridge rules that invoke programmatic actions and notifications in response to a variety of these events\. For example, while instances are in the process of launching or terminating, you can invoke an AWS Lambda function to perform a preconfigured task\.

Targets of EventBridge rules can include AWS Lambda functions, Amazon SNS topics, API destinations, event buses in other AWS accounts, and many more\. For information about supported targets, see [Amazon EventBridge targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-targets.html) in the *Amazon EventBridge User Guide*\.

Get started by creating EventBridge rules with an example using an Amazon SNS topic and an EventBridge rule\. Then, when a user starts an instance refresh, Amazon SNS notifies you by email whenever a checkpoint is reached\. For more information, see [Create EventBridge rules for instance refresh events](monitor-events-eventbridge-sns.md)\. 

**Topics**
+ [Amazon EC2 Auto Scaling event reference](ec2-auto-scaling-event-reference.md)
+ [Warm pool event types and patterns](warm-pools-eventbridge-events.md)
+ [Create EventBridge rules](create-eventbridge-rules.md)