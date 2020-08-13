# Automating Amazon EC2 Auto Scaling with EventBridge<a name="cloud-watch-events"></a>

Amazon EventBridge, formerly called CloudWatch Events, lets you automate AWS services and respond to system events such as application availability issues or resource changes\. Events from AWS services are delivered to EventBridge in near real time\. Based on the rules that you create, EventBridge invokes one or more target actions when an event matches the values that you specify in a rule\. The actions that can be automatically triggered include the following: 
+ Invoking an AWS Lambda function
+ Invoking Amazon EC2 Run Command
+ Relaying the event to Amazon Kinesis Data Streams
+ Activating an AWS Step Functions state machine
+ Notifying an Amazon SNS topic or an Amazon SQS queue 

As an example of a situation in which EventBridge can be useful, you might invoke a Lambda function whenever your Auto Scaling group scales\. First, create your Lambda function, then create an EventBridge rule that triggers on events emitted by Amazon EC2 Auto Scaling, as described in the following sections\. For an example of automation that you can create when a lifecycle action occurs, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

You can also create a rule that triggers on an Amazon EC2 Auto Scaling API call via CloudTrail\. For more information, see [Creating an EventBridge rule that triggers on an AWS API call using AWS CloudTrail](https://docs.aws.amazon.com/eventbridge/latest/userguide/create-eventbridge-cloudtrail-rule.html) in the *Amazon EventBridge User Guide*\. 

For more information, see the [Amazon EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/)\.

**Note**  
When Amazon EC2 is going to interrupt your Spot Instance, it emits an event two minutes prior to the actual interruption\. You can also create an EventBridge rule to capture these events\. For more information, see [Spot instance interruption notices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#spot-instance-termination-notices) in the *Amazon EC2 User Guide for Linux Instances*\.

**Contents**
+ [Auto Scaling events](#cloudwatch-event-types)
  + [EC2 instance\-launch lifecycle action](#launch-lifecycle-action)
  + [EC2 instance launch successful](#launch-successful)
  + [EC2 instance launch unsuccessful](#launch-unsuccessful)
  + [EC2 instance\-terminate lifecycle action](#terminate-lifecycle-action)
  + [EC2 instance terminate successful](#terminate-successful)
  + [EC2 instance terminate unsuccessful](#terminate-unsuccessful)
+ [Create a Lambda function](#create-lambda-function)
+ [Route events to your Lambda function](#create-rule)

## Auto Scaling events<a name="cloudwatch-event-types"></a>

You can configure EventBridge to send events to the configured target when the following events occur: 

### EC2 instance\-launch lifecycle action<a name="launch-lifecycle-action"></a>

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

### EC2 instance launch successful<a name="launch-successful"></a>

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

### EC2 instance launch unsuccessful<a name="launch-unsuccessful"></a>

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

### EC2 instance\-terminate lifecycle action<a name="terminate-lifecycle-action"></a>

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

### EC2 instance terminate successful<a name="terminate-successful"></a>

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

### EC2 instance terminate unsuccessful<a name="terminate-unsuccessful"></a>

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

## Create a Lambda function<a name="create-lambda-function"></a>

Use the following procedure to create a Lambda function using the **hello\-world** blueprint to serve as the target for events\.

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

   1. For **Role**, choose **Choose an existing role**\. For **Existing role**, select your basic execution role\. Otherwise, create a basic execution role\.

   1. \(Optional\) For **Advanced settings**, make any changes that you need\.

   1. Choose **Next**\.

1. On the **Review** page, choose **Create function**\.

## Route events to your Lambda function<a name="create-rule"></a>

Create a rule that matches selected events and route them to your Lambda function to take action\. 

**To create a rule that routes events to your Lambda function**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, choose **Rules**\.

1. Choose **Create rule**\.

1. Enter a name and description for the rule\.

1. For **Define pattern**, do the following:

   1. Choose **Event Pattern**\.

   1. Choose **Pre\-defined by service**\.

   1. For **Service provider**, choose **AWS**\.

   1. For **Service Name**, choose **Auto Scaling**\.

   1. For **Event type**, choose **Instance Launch and Terminate**\.

   1. To capture all successful and unsuccessful instance launch and terminate events, choose **Any instance event**\.

   1. By default, the rule matches any Auto Scaling group in the Region\. To make the rule match a specific Auto Scaling group, choose **Specific group name\(s\)** and select one or more Auto Scaling groups\.

1. For **Select event bus**, choose **AWS default event bus**\. When an AWS service in your account emits an event, it always goes to your account's default event bus\. 

1. For **Target**, choose **Lambda function**\.

1. For **Function**, select the Lambda function that you created\.

1. Choose **Create**\.

To test your rule, change the size of your Auto Scaling group\. If you used the example code for your Lambda function, it logs the event to CloudWatch Logs\.

**To test your rule**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**, and then select your Auto Scaling group\.

1. On the **Details** tab, choose **Edit** from the right side of the page\.

1. Change the value of **Desired capacity**, and then choose **Update**\.

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation pane, choose **Logs**\.

1. Select the log group for your Lambda function \(for example, **/aws/lambda/***my\-function*\)\.

1. Select a log stream to view the event data\. The data is displayed, similar to the following:  
![\[Viewing event data for Amazon EC2 Auto Scaling in CloudWatch Logs.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/auto_scaling_event_data.png)