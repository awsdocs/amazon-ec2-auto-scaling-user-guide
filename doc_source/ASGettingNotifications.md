# Getting Amazon SNS Notifications When Your Auto Scaling Group Scales<a name="ASGettingNotifications"></a>

It is useful to know when Amazon EC2 Auto Scaling is launching or terminating the EC2 instances in your Auto Scaling group\. Amazon SNS coordinates and manages the delivery or sending of notifications to subscribing clients or endpoints\. You can configure Amazon EC2 Auto Scaling to send an SNS notification whenever your Auto Scaling group scales\.

 Amazon SNS can deliver notifications as HTTP or HTTPS POST, email \(SMTP, either plaintext or in JSON format\), or as a message posted to an Amazon SQS queue\. For more information, see [What Is Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/) in the *Amazon Simple Notification Service Developer Guide*\.

For example, if you configure your Auto Scaling group to use the `autoscaling: EC2_INSTANCE_TERMINATE` notification type, and your Auto Scaling group terminates an instance, it sends an email notification\. This email contains the details of the terminated instance, such as the instance ID and the reason that the instance was terminated\.

**Tip**  
If you prefer, you can use Amazon CloudWatch Events to configure a target to invoke a Lambda function when your Auto Scaling group scales or when a lifecycle action occurs\. For more information, see [Getting CloudWatch Events When Your Auto Scaling Group Scales](cloud-watch-events.md)\.

**Topics**
+ [SNS Notifications](#auto-scaling-sns-notifications)
+ [Configure Amazon SNS](#as-configure-sns)
+ [Configure Your Auto Scaling Group to Send Notifications](#as-configure-asg-for-sns)
+ [Test the Notification Configuration](#update-settingupnotifications)
+ [Verify That You Received Notification of the Scaling Event](#as-verify-sns-notification)
+ [Delete the Notification Configuration](#delete-settingupnotifications)

## SNS Notifications<a name="auto-scaling-sns-notifications"></a>

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
AutoScalingGroupARN: arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup...
ActivityId: 4e6156f4-a9e2-4bda-a7fd-33f2ae528958
Description: Launching a new EC2 instance: i-0598c7d356eba48d7
Cause: At 2016-09-30T18:59:38Z a user request update of AutoScalingGroup constraints to ...
StartTime: 2016-09-30T19:00:04.445Z
EndTime: 2016-09-30T19:00:36.414Z
StatusCode: InProgress
StatusMessage: 
Progress: 50
EC2InstanceId: i-0598c7d356eba48d7
Details: {"Subnet ID":"subnet-c9663da0","Availability Zone":"us-west-2b"}
```

## Configure Amazon SNS<a name="as-configure-sns"></a>

To use Amazon SNS to send email notifications, you must first create a *topic* and then subscribe your email addresses to the topic\.

### Create an Amazon SNS Topic<a name="as-sns-create-topic"></a>

 An SNS topic is a logical access point, a communication channel your Auto Scaling group uses to send the notifications\. You create a topic by specifying a name for your topic\.

For more information, see [Create a Topic](https://docs.aws.amazon.com/sns/latest/dg/CreateTopic.html) in the *Amazon Simple Notification Service Developer Guide*\.

### Subscribe to the Amazon SNS Topic<a name="as-sns-subscribe-topic"></a>

To receive the notifications that your Auto Scaling group sends to the topic, you must subscribe an endpoint to the topic\. In this procedure, for **Endpoint**, specify the email address where you want to receive the notifications from Amazon EC2 Auto Scaling\.

For more information, see [Subscribe to a Topic](https://docs.aws.amazon.com/sns/latest/dg/SubscribeTopic.html) in the *Amazon Simple Notification Service Developer Guide*\.

### Confirm Your Amazon SNS Subscription<a name="as-sns-confirm-subscription"></a>

Amazon SNS sends a confirmation email to the email address you specified in the previous step\.

Make sure that you open the email from AWS Notifications and choose the link to confirm the subscription before you continue with the next step\.

You will receive an acknowledgement message from AWS\. Amazon SNS is now configured to receive notifications and send the notification as an email to the email address that you specified\.

## Configure Your Auto Scaling Group to Send Notifications<a name="as-configure-asg-for-sns"></a>

You can configure your Auto Scaling group to send notifications to Amazon SNS when a scaling event, such as launching instances or terminating instances, takes place\. Amazon SNS sends a notification with information about the instances to the email address that you specified\.

**To configure Amazon SNS notifications for your Auto Scaling group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Notifications** tab, choose **Create notification**\.

1. On the **Create notifications** pane, do the following:

   1. For **Send a notification to:**, select your SNS topic\.

   1. For **Whenever instances**, select the events to send the notifications for\.

   1. Choose **Save**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-add-notification.png)

**To configure Amazon SNS notifications for your Auto Scaling group using the AWS CLI**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-notification-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-notification-configuration.html) command:

```
aws autoscaling put-notification-configuration --auto-scaling-group-name my-asg --topic-arn arn --notification-types "autoscaling:EC2_INSTANCE_LAUNCH" "autoscaling:EC2_INSTANCE_TERMINATE"
```

## Test the Notification Configuration<a name="update-settingupnotifications"></a>

To generate a notification for a launch event, update the Auto Scaling group by increasing the desired capacity of the Auto Scaling group by 1\. Amazon EC2 Auto Scaling launches the EC2 instance, and you receive an email notification within a few minutes\.

**To change the desired capacity using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Details** tab, choose **Edit**\.

1. For **Desired**, increase the current value by 1\. If this value exceeds **Max**, you must also increase the value of **Max** by 1\.

1. Choose **Save**\.

1. After a few minutes, you'll receive a notification email for the launch event\. If you do not need the additional instance that you launched for this test, you can decrease **Desired** by 1\. After a few minutes, you'll receive a notification email for the terminate event\.

**To change the desired capacity using the AWS CLI**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html) command:

```
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg --desired-capacity 2
```

## Verify That You Received Notification of the Scaling Event<a name="as-verify-sns-notification"></a>

Check your email for a message from Amazon SNS and open the email\. After you receive notification of a scaling event for your Auto Scaling group, you can confirm the scaling event by looking at the description of your Auto Scaling group\. You need information from the notification email, such as the ID of the instance that was launched or terminated\.

**To verify that your Auto Scaling group has launched new instance using the console**

1. Select your Auto Scaling group\.

1. On the **Activity History** tab, the **Status** column shows the current status of your instance\. For example, if the notification indicates that an instance has launched, use the refresh button to verify that the status of the launch activity is **Successful**\.

1. On the **Instances** tab, you can view the current **Lifecycle** state of the instance whose ID you received in the notification email\. After a new instance starts, its lifecycle state changes to `InService`\.

**To verify that your Auto Scaling group has launched a new instance using the AWS CLI**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to confirm that the size of your Auto Scaling group has changed:

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
```

The following example output shows that the group has two instances\. Check for the instance whose ID you received in the notification email\.

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupARN": "arn",
            "HealthCheckGracePeriod": 0,
            "SuspendedProcesses": [],
            "DesiredCapacity": 2,
            "Tags": [],
            "EnabledMetrics": [],
            "LoadBalancerNames": [],
            "AutoScalingGroupName": "my-asg",
            "DefaultCooldown": 300,
            "MinSize": 1,
            "Instances": [
                {
                    "InstanceId": "i-d95eb0d4",
                    "AvailabilityZone": "us-west-2b",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService",
                    "LaunchConfigurationName": "my-lc"
                },
                {
                    "InstanceId": "i-13d7dc1f",
                    "AvailabilityZone": "us-west-2a",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService",
                    "LaunchConfigurationName": "my-lc"
                }
            ],
            "MaxSize": 5,
            "VPCZoneIdentifier": null,
            "TerminationPolicies": [
                "Default"
            ],
            "LaunchConfigurationName": "my-lc",
            "CreatedTime": "2015-03-01T16:12:35.608Z",
            "AvailabilityZones": [
                "us-west-2b",
                "us-west-2a"
            ],
            "HealthCheckType": "EC2"
        }
    ]
}
```

## Delete the Notification Configuration<a name="delete-settingupnotifications"></a>

You can delete your Amazon EC2 Auto Scaling notification configuration at any time\.

**To delete Amazon EC2 Auto Scaling notification configuration using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Notifications** tab, choose **Delete** next to the notification\.

**To delete Amazon EC2 Auto Scaling notification configuration using the AWS CLI**  
Use the following delete\-notification\-configuration command:

```
aws autoscaling delete-notification-configuration --auto-scaling-group-name my-asg --topic-arn arn:aws:sns:us-west-2:123456789012:my-sns-topic
```

For information about deleting the Amazon SNS topic and all subscriptions associated with your Auto Scaling group, see [Clean Up](https://docs.aws.amazon.com/sns/latest/dg/CleanUp.html) in the *Amazon Simple Notification Service Developer Guide*\.