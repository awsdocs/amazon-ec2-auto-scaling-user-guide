# Create EventBridge rules for instance refresh events<a name="monitor-events-eventbridge-sns"></a>

This section shows you how to create an Amazon EventBridge rule that notifies you whenever a checkpoint is reached during an instance refresh\. The procedure for setting up email notifications through Amazon SNS is included\. To use Amazon SNS to send email notifications, you must first create a *topic* and then subscribe your email addresses to the topic\.

For more information about working with EventBridge, see [Use EventBridge to handle Auto Scaling events](automating-ec2-auto-scaling-with-eventbridge.md)\.

## Create an Amazon SNS topic<a name="eventbridge-sns-create-topic"></a>

An SNS topic is a logical access point, a communication channel that your Auto Scaling group uses to send the notifications\. You create a topic by specifying a name for your topic\.

When you create a topic name, the name must meet the following requirements:
+ Between 1 and 256 characters long
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

**To create a rule that routes instance refresh events to your Amazon SNS topic**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, under **Events**, choose **Rules**\.

1. In the **Rules** section, choose **Create rule**\.

1. Enter a name and description for the rule\.

1. For **Define pattern**, do the following:

   1. Choose **Event Pattern**\.

   1. For **Event matching pattern**, choose **Pre\-defined by service**\.

   1. For **Service provider**, choose **Amazon Web Services**\.

   1. For **Service Name**, choose **Auto Scaling**\.

   1. For **Event type**, choose **Instance Refresh**\.

   1. By default, the rule matches any instance refresh event\. To create a rule that notifies you whenever a checkpoint is reached during an instance refresh, choose **Specific instance event\(s\)** and select **EC2 Auto Scaling Instance Refresh Checkpoint Reached**\.

   1. By default, the rule matches any Auto Scaling group in the Region\. To make the rule match a specific Auto Scaling group, choose **Specific group name\(s\)** and select one or more Auto Scaling groups\.

1. For **Select event bus**, choose **AWS default event bus**\. When an AWS service in your account emits an event, it always goes to your account's default event bus\. 

1. For **Target**, choose **SNS topic**\.

1. For **Topic**, select the Amazon SNS topic that you created\.

1. For **Configure input**, choose the input for the email notification\. 

1. Choose **Create**\.