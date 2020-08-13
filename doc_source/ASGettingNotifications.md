# Getting Amazon SNS notifications when your Auto Scaling group scales<a name="ASGettingNotifications"></a>

You can be notified when Amazon EC2 Auto Scaling is launching or terminating the EC2 instances in your Auto Scaling group\. You manage notifications using Amazon Simple Notification Service \(Amazon SNS\)\. 

Amazon SNS coordinates and manages the delivery or sending of notifications to subscribing clients or endpoints\. Amazon SNS offers a variety of notification options, including the ability to deliver notifications as HTTP or HTTPS POST, email \(SMTP, either plaintext or in JSON format\), or as a message posted to an Amazon SQS queue, which enables you to handle these notifications programmatically\. For more information, see [Amazon Simple Notification Service Developer Guide](https://docs.aws.amazon.com/sns/latest/dg/)\.

For example, if you configure your Auto Scaling group to use the `autoscaling: EC2_INSTANCE_TERMINATE` notification type, and your Auto Scaling group terminates an instance, it sends an email notification\. This email contains the details of the terminated instance, such as the instance ID and the reason that the instance was terminated\.

Notifications are useful for designing event\-driven applications\. If you use notifications to check that a resource enters a desired state, you can eliminate polling, and you won't encounter the `RequestLimitExceeded` error that sometimes results from polling\. 

AWS provides various tools that you can use to send notifications\. Alternatively, you can use EventBridge and Amazon SNS to send notifications when your Auto Scaling groups launch or terminate instances\. In EventBridge, the rule describes which events you're notified about\. In Amazon SNS, the topic describes what kind of notification you receive\. Using this option, you can decide if certain events should trigger a Lambda function instead\. For more information, see [Automating Amazon EC2 Auto Scaling with EventBridge](cloud-watch-events.md)\. 

**Contents**
+ [SNS notifications](#auto-scaling-sns-notifications)
+ [Configuring Amazon SNS notifications for Amazon EC2 Auto Scaling](#as-configure-sns)
  + [Create an Amazon SNS topic](#as-sns-create-topic)
  + [Subscribe to the Amazon SNS topic](#as-sns-subscribe-topic)
  + [Confirm your Amazon SNS subscription](#as-sns-confirm-subscription)
  + [Configure your Auto Scaling group to send notifications](#as-configure-asg-for-sns)
  + [Test the notification](#testing-hook-notifications)
  + [Delete the notification configuration](#delete-settingupnotifications)

## SNS notifications<a name="auto-scaling-sns-notifications"></a>

Amazon EC2 Auto Scaling supports sending Amazon SNS notifications when the following events occur\.


| Event | Description | 
| --- | --- | 
|  `autoscaling:EC2_INSTANCE_LAUNCH`  | Successful instance launch | 
|  `autoscaling:EC2_INSTANCE_LAUNCH_ERROR`  | Failed instance launch | 
|  `autoscaling:EC2_INSTANCE_TERMINATE`  | Successful instance termination | 
|  `autoscaling:EC2_INSTANCE_TERMINATE_ERROR`  | Failed instance termination | 

The message includes the following information:
+ `Event` — The event\.
+ `AccountId` — The AWS account ID\.
+ `AutoScalingGroupName` — The name of the Auto Scaling group\.
+ `AutoScalingGroupARN` — The ARN of the Auto Scaling group\.
+ `EC2InstanceId` — The ID of the EC2 instance\.

For example:

```
Service: AWS Auto Scaling
Time: 2016-09-30T19:00:36.414Z
RequestId: 4e6156f4-a9e2-4bda-a7fd-33f2ae528958
Event: autoscaling:EC2_INSTANCE_LAUNCH
AccountId: 123456789012
AutoScalingGroupName: my-asg
AutoScalingGroupARN: arn:aws:autoscaling:region:123456789012:autoScalingGroup...
ActivityId: 4e6156f4-a9e2-4bda-a7fd-33f2ae528958
Description: Launching a new EC2 instance: i-0598c7d356eba48d7
Cause: At 2016-09-30T18:59:38Z a user request update of AutoScalingGroup constraints to ...
StartTime: 2016-09-30T19:00:04.445Z
EndTime: 2016-09-30T19:00:36.414Z
StatusCode: InProgress
StatusMessage: 
Progress: 50
EC2InstanceId: i-0598c7d356eba48d7
Details: {"Subnet ID":"subnet-id","Availability Zone":"zone"}
```

## Configuring Amazon SNS notifications for Amazon EC2 Auto Scaling<a name="as-configure-sns"></a>

To use Amazon SNS to send email notifications, you must first create a *topic* and then subscribe your email addresses to the topic\.

### Create an Amazon SNS topic<a name="as-sns-create-topic"></a>

 An SNS topic is a logical access point, a communication channel your Auto Scaling group uses to send the notifications\. You create a topic by specifying a name for your topic\.

When you create a topic name, the name must meet the following requirements:
+ Between 1 and 256 characters long
+ Contain uppercase and lowercase ASCII letters, numbers, underscores, or hyphens 

For more information, see [Tutorial: Creating an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.

### Subscribe to the Amazon SNS topic<a name="as-sns-subscribe-topic"></a>

To receive the notifications that your Auto Scaling group sends to the topic, you must subscribe an endpoint to the topic\. In this procedure, for **Endpoint**, specify the email address where you want to receive the notifications from Amazon EC2 Auto Scaling\.

For more information, see [Tutorial: Subscribing an endpoint to an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-subscribe-endpoint-to-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.

### Confirm your Amazon SNS subscription<a name="as-sns-confirm-subscription"></a>

Amazon SNS sends a confirmation email to the email address you specified in the previous step\.

Make sure that you open the email from AWS Notifications and choose the link to confirm the subscription before you continue with the next step\.

You will receive an acknowledgment message from AWS\. Amazon SNS is now configured to receive notifications and send the notification as an email to the email address that you specified\.

### Configure your Auto Scaling group to send notifications<a name="as-configure-asg-for-sns"></a>

You can configure your Auto Scaling group to send notifications to Amazon SNS when a scaling event, such as launching instances or terminating instances, takes place\. Amazon SNS sends a notification with information about the instances to the email address that you specified\.

**To configure Amazon SNS notifications for your Auto Scaling group \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Activity** tab, choose **Activity notifications**, **Create notification**\. \(Old console: On the **Notifications** tab, choose **Create notification**\.\)

1. On the **Create notifications** pane, do the following:

   1. For **SNS Topic**, select your SNS topic\. \(Old console: For **Send a notification to:**, select your SNS topic\.\)

   1. For **Event types**, select the events to send the notifications\. \(Old console: For **Whenever instances**, select the events to send the notifications for\.\)

   1. Choose **Create**\.

**To configure Amazon SNS notifications for your Auto Scaling group \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-notification-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-notification-configuration.html) command\.

```
aws autoscaling put-notification-configuration --auto-scaling-group-name my-asg --topic-arn arn --notification-types "autoscaling:EC2_INSTANCE_LAUNCH" "autoscaling:EC2_INSTANCE_TERMINATE"
```

### Test the notification<a name="testing-hook-notifications"></a>

To generate a notification for a launch event, update the Auto Scaling group by increasing the desired capacity of the Auto Scaling group by 1\. You receive a notification within a few minutes after instance launch\.

**To change the desired capacity \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Group details**, **Edit**\. \(Old console: On the **Details** tab, choose **Edit**\.\)

1. For **Desired capacity**, increase the current value by 1\. If this value exceeds **Maximum capacity**, you must also increase the value of **Maximum capacity** by 1\.

1. Choose **Update**\.

1. After a few minutes, you'll receive notification for the event\. If you do not need the additional instance that you launched for this test, you can decrease **Desired capacity** by 1\. After a few minutes, you'll receive notification for the event\.

### Delete the notification configuration<a name="delete-settingupnotifications"></a>

You can delete your Amazon EC2 Auto Scaling notification configuration if it is no longer being used\.

**To delete Amazon EC2 Auto Scaling notification configuration \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Activity** tab, select the check box next to the notification you want to delete and then choose **Actions**, **Delete**\. \(Old console: On the **Notifications** tab, choose **Delete** next to the notification\.\)

**To delete Amazon EC2 Auto Scaling notification configuration \(AWS CLI\)**  
Use the following delete\-notification\-configuration command\.

```
aws autoscaling delete-notification-configuration --auto-scaling-group-name my-asg --topic-arn arn
```

For information about deleting the Amazon SNS topic and all subscriptions associated with your Auto Scaling group, see [Tutorial: Deleting an Amazon SNS subscription and topic](https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-delete-subscription-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.