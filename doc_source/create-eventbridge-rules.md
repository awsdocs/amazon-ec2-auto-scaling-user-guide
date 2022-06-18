# Create EventBridge rules<a name="create-eventbridge-rules"></a>

When an event is emitted by Amazon EC2 Auto Scaling, an event notification is sent to Amazon EventBridge as a JSON file\. You can write an EventBridge rule to automate what actions to take when an event pattern matches the rule\. If EventBridge detects an event pattern that matches a pattern defined in a rule, EventBridge invokes the target \(or targets\) specified in the rule\. 

You can use the example procedures in this section as a starting point\.

You may also find the following documentation useful\.
+ To perform custom actions on instances as they are launching or before they are terminated using a Lambda function, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\.
+ To invoke a Lambda function on API calls logged with CloudTrail, see [Tutorial: Log AWS API calls using EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-log-api-call.html) in the *Amazon EventBridge User Guide*\.
+ For more information about how to create event rules, see [Creating Amazon EventBridge rules that react to events](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule.html) in the *Amazon EventBridge User Guide*\.

**Topics**
+ [Create EventBridge rules for instance refresh events](monitor-events-eventbridge-sns.md)
+ [Create EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md)