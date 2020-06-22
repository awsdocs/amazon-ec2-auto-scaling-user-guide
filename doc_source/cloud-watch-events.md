# Getting CloudWatch Events When Your Auto Scaling Group Scales<a name="cloud-watch-events"></a>

Amazon CloudWatch Events lets you automate AWS services and respond to system events such as application availability issues or resource changes\. Events from AWS services are delivered to CloudWatch Events nearly in real time\. You can write simple rules to indicate which events are of interest to you and what automated actions to take when an event matches a rule\.

CloudWatch Events lets you set a variety of targets—such as a Lambda function or an Amazon SNS topic—which receive events in JSON format\. For more information, see the [Amazon CloudWatch Events User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)\.

It is useful to know when Amazon EC2 Auto Scaling is launching or terminating the EC2 instances in your Auto Scaling group\. You can configure Amazon EC2 Auto Scaling to send events to CloudWatch Events whenever your Auto Scaling group scales\.

**Note**  
You can also receive a two\-minute warning when Spot Instances are about to be reclaimed by Amazon EC2\. For an example of the event for Spot Instance interruption, see [Spot Instance Interruption Notices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#spot-instance-termination-notices) in the *Amazon EC2 User Guide for Linux Instances*\.

**Topics**
+ [Auto Scaling Events](#cloudwatch-event-types)
+ [Create a Lambda Function](#create-lambda-function)
+ [Route Events to Your Lambda Function](#create-rule)

## Auto Scaling Events<a name="cloudwatch-event-types"></a>

Amazon EC2 Auto Scaling supports sending events to CloudWatch Events when the following events occur:
+ [EC2 Instance\-launch Lifecycle Action](#launch-lifecycle-action)
+ [EC2 Instance Launch Successful](#launch-successful)
+ [EC2 Instance Launch Unsuccessful](#launch-unsuccessful)
+ [EC2 Instance\-terminate Lifecycle Action](#terminate-lifecycle-action)
+ [EC2 Instance Terminate Successful](#terminate-successful)
+ [EC2 Instance Terminate Unsuccessful](#terminate-unsuccessful)

### EC2 Instance\-launch Lifecycle Action<a name="launch-lifecycle-action"></a>

Amazon EC2 Auto Scaling moved an instance to a `Pending:Wait` state due to a lifecycle hook\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance-launch Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": { 
    "LifecycleActionToken": "87654321-4321-4321-4321-210987654321", 
    "AutoScalingGroupName": "my-asg", 
    "LifecycleHookName": "my-lifecycle-hook", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info"
  } 
}
```

### EC2 Instance Launch Successful<a name="launch-successful"></a>

Amazon EC2 Auto Scaling successfully launched an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Launch Successful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
      "auto-scaling-group-arn",
      "instance-arn"
  ],
  "detail": {
      "StatusCode": "InProgress",
      "Description": "Launching a new EC2 instance: i-12345678",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

### EC2 Instance Launch Unsuccessful<a name="launch-unsuccessful"></a>

Amazon EC2 Auto Scaling failed to launch an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Launch Unsuccessful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn",
    "instance-arn"
  ],
  "detail": {
      "StatusCode": "Failed",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "message-text",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

### EC2 Instance\-terminate Lifecycle Action<a name="terminate-lifecycle-action"></a>

Amazon EC2 Auto Scaling moved an instance to a `Terminating:Wait` state due to a lifecycle hook\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance-terminate Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": { 
    "LifecycleActionToken":"87654321-4321-4321-4321-210987654321", 
    "AutoScalingGroupName":"my-asg", 
    "LifecycleHookName":"my-lifecycle-hook", 
    "EC2InstanceId":"i-1234567890abcdef0", 
    "LifecycleTransition":"autoscaling:EC2_INSTANCE_TERMINATING", 
    "NotificationMetadata":"additional-info"
  } 
}
```

### EC2 Instance Terminate Successful<a name="terminate-successful"></a>

Amazon EC2 Auto Scaling successfully terminated an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Terminate Successful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn",
    "instance-arn"
  ],
  "detail": {
      "StatusCode": "InProgress",
      "Description": "Terminating EC2 instance: i-12345678",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

### EC2 Instance Terminate Unsuccessful<a name="terminate-unsuccessful"></a>

Amazon EC2 Auto Scaling failed to terminate an instance\.

**Event Data**  
The following is example data for this event\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance Terminate Unsuccessful",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn",
    "instance-arn"
  ],
  "detail": {
      "StatusCode": "Failed",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "ActivityId": "87654321-4321-4321-4321-210987654321",
      "Details": {
          "Availability Zone": "us-west-2b",
          "Subnet ID": "subnet-12345678"
      },
      "RequestId": "12345678-1234-1234-1234-123456789012",
      "StatusMessage": "message-text",
      "EndTime": "yyyy-mm-ddThh:mm:ssZ",
      "EC2InstanceId": "i-1234567890abcdef0",
      "StartTime": "yyyy-mm-ddThh:mm:ssZ",
      "Cause": "description-text"
  }
}
```

## Create a Lambda Function<a name="create-lambda-function"></a>

Use the following procedure to create a Lambda function to handle an Auto Scaling event\.

**To create a Lambda function**

1. Open the AWS Lambda console at [https://console\.aws\.amazon\.com/lambda/](https://console.aws.amazon.com/lambda/)\.

1. If you are new to Lambda, you see a welcome page; choose **Get Started Now**; otherwise, choose **Create a Lambda function**\.

1. On the **Select blueprint** page, enter `hello-world` for **Filter**, and then select the **hello\-world** blueprint\.

1. On the **Configure triggers** page, choose **Next**\.

1. On the **Configure function** page, do the following:

   1. Enter a name and description for the Lambda function\.

   1. Edit the code for the Lambda function\. For example, the following code simply logs the event\.

      ```
      console.log('Loading function');
      
      exports.handler = function(event, context) {
          console.log("AutoScalingEvent()");
          console.log("Event data:\n" + JSON.stringify(event, null, 4));
          context.succeed("...");
      };
      ```

   1. For **Role**, choose **Choose an existing role** if you have an existing role that you'd like to use, and then choose your role from **Existing role**\. Alternatively, to create a new role, choose one of the other options for **Role** and then follow the directions\.

   1. \(Optional\) For **Advanced settings**, make any changes that you need\.

   1. Choose **Next**\.

1. On the **Review** page, choose **Create function**\.

## Route Events to Your Lambda Function<a name="create-rule"></a>

Use the following procedure to route Auto Scaling events to your Lambda function\.

**To route events to your Lambda function**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation pane, choose **Events**\.

1. Choose **Create rule**\.

1. For **Event selector**, choose **Auto Scaling** as the event source\. By default, the rule applies to all Auto Scaling events for all of your Auto Scaling groups\. Alternatively, you can select specific events or a specific Auto Scaling group\.

1. For **Targets**, choose **Add target**\. Choose **Lambda function** as the target type, and then select your Lambda function\.

1. Choose **Configure details**\.

1. For **Rule definition**, enter a name and description for your rule\.

1. Choose **Create rule**\.

To test your rule, change the size of your Auto Scaling group\. If you used the example code for your Lambda function, it logs the event to CloudWatch Logs\.

**To test your rule**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**, and then select your Auto Scaling group\.

1. On the **Details** tab, choose **Edit**\.

1. Change the value of **Desired**, and then choose **Save**\.

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation pane, choose **Logs**\.

1. Select the log group for your Lambda function \(for example, **/aws/lambda/***my\-function*\)\.

1. Select a log stream to view the event data\. The data is displayed, similar to the following:  
![\[Viewing event data for Amazon EC2 Auto Scaling in CloudWatch Logs.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/auto_scaling_event_data.png)