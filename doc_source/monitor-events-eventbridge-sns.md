# Create EventBridge rules for instance refresh events<a name="monitor-events-eventbridge-sns"></a>

The following example creates an EventBridge rule to send an email notification\. It does this each time that your Auto Scaling group emits an event when a checkpoint is reached during an instance refresh\. The procedure for setting up email notifications using Amazon SNS is included\. To use Amazon SNS to send email notifications, you must first create a *topic* and then subscribe your email addresses to the topic\.

For more information about the instance refresh feature, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

## Create an Amazon SNS topic<a name="eventbridge-sns-create-topic"></a>

An SNS topic is a logical access point, a communication channel that your Auto Scaling group uses to send the notifications\. You create a topic by specifying a name for your topic\.

Topic names must meet the following requirements:
+ Have 1\-256 characters
+ Contain uppercase and lowercase ASCII letters, numbers, underscores, or hyphens 

For more information, see [Creating an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.

## Subscribe to the Amazon SNS topic<a name="eventbridge-sns-subscribe-topic"></a>

To receive the notifications that your Auto Scaling group sends to the topic, you must subscribe an endpoint to the topic\. In this procedure, for **Endpoint**, specify the email address where you want to receive the notifications from Amazon EC2 Auto Scaling\.

For more information, see [Subscribing to an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-subscribe-endpoint-to-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.

## Confirm your Amazon SNS subscription<a name="eventbridge-sns-confirm-subscription"></a>

Amazon SNS sends a confirmation email to the email address you specified in the previous step\.

Make sure that you open the email from AWS Notifications and choose the link to confirm the subscription before you continue with the next step\.

You will receive an acknowledgment message from AWS\. Amazon SNS is now configured to receive notifications and send the notification as an email to the email address that you specified\.

## Route events to your Amazon SNS topic<a name="eventbridge-sns-create-rule"></a>

Create a rule that matches selected events and routes them to your Amazon SNS topic to notify subscribed email addresses\.

**To create a rule that sends notifications to your Amazon SNS topic**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, choose **Rules**\.

1. Choose **Create rule**\.

1. For **Define rule detail**, do the following:

   1. Enter a **Name** for the rule, and, optionally, a description\.

      A rule can't have the same name as another rule in the same Region and on the same event bus\.

   1. For **Event bus**, choose **default**\. When an AWS service in your account generates an event, it always goes to your account's default event bus\.

   1. For **Rule type**, choose **Rule with an event pattern**\.

   1. Choose **Next**\.

1. For **Build event pattern**, do the following:

   1. For **Event source**, choose **AWS events or EventBridge partner events**\.

   1. For **Event pattern**, do the following:

      1. For **Event source**, choose **AWS services**\.

      1. For **AWS service**, choose **Auto Scaling**\.

      1. For **Event type**, choose **Instance Refresh**\.

      1. By default, the rule matches any instance refresh event\. To create a rule that notifies you when a checkpoint is reached during an instance refresh, choose **Specific instance event\(s\)** and select **EC2 Auto Scaling Instance Refresh Checkpoint Reached**\.

      1. By default, the rule matches any Auto Scaling group in the Region\. To make the rule match a specific Auto Scaling group, choose **Specific group name\(s\)** and select one or more Auto Scaling groups\.

      1. Choose **Next**\.

1. For **Select target\(s\)**, do the following:

   1. For **Target types**, choose **AWS service**\.

   1. For **Select a target**, choose **SNS topic**\.

   1. For **Topic**, choose your Amazon SNS topic\.

   1. \(Optional\) Under **Additional settings**, you can optionally configure additional settings\. For more information, see [Creating Amazon EventBridge rules that react to events](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule.html) \(step 16\) in the *Amazon EventBridge User Guide*\.

   1. Choose **Next**\.

1. \(Optional\) For **Tags**, you can optionally assign one or more tags to your rule, and then choose **Next**\.

1. For **Review and create**, review the details of the rule and modify them as necessary\. Then, choose **Create rule**\.