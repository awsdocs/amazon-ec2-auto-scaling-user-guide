# Amazon EC2 Auto Scaling event reference<a name="ec2-auto-scaling-event-reference"></a>

Using Amazon EventBridge, you can create *rules* that match incoming *events* and route them to *targets* for processing\. 

## Example events<a name="ec2-auto-scaling-example-event-types"></a>

The following examples show events for Amazon EC2 Auto Scaling\. Events are produced on a best\-effort basis\.

**Topics**
+ [EC2 Instance\-launch Lifecycle Action](#launch-lifecycle-action)
+ [EC2 Instance Launch Successful](#launch-successful)
+ [EC2 Instance Launch Unsuccessful](#launch-unsuccessful)
+ [EC2 Instance\-terminate Lifecycle Action](#terminate-lifecycle-action)
+ [EC2 Instance Terminate Successful](#terminate-successful)
+ [EC2 Instance Terminate Unsuccessful](#terminate-unsuccessful)
+ [EC2 Auto Scaling Instance Refresh Checkpoint Reached](#instance-refresh-checkpoint-reached)
+ [EC2 Auto Scaling Instance Refresh Started](#instance-refresh-started)
+ [EC2 Auto Scaling Instance Refresh Succeeded](#instance-refresh-succeeded)
+ [EC2 Auto Scaling Instance Refresh Failed](#instance-refresh-failed)
+ [EC2 Auto Scaling Instance Refresh Cancelled](#instance-refresh-cancelled)

### EC2 Instance\-launch Lifecycle Action<a name="launch-lifecycle-action"></a>

In this example event, Amazon EC2 Auto Scaling moved an instance to a `Pending:Wait` state due to a lifecycle hook\. 

**Note**  
Amazon EC2 Auto Scaling supports providing `Origin` and `Destination` in events for lifecycle hooks, if that information is available\. For more information, see [Warm pool event types and patterns](warm-pools-eventbridge-events.md)\.

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

In this example event, Amazon EC2 Auto Scaling successfully launched an instance\.

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

In this example event, Amazon EC2 Auto Scaling failed to launch an instance\.

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

In this example event, Amazon EC2 Auto Scaling moved an instance to a `Terminating:Wait` state due to a lifecycle hook\.

**Note**  
Amazon EC2 Auto Scaling supports providing `Origin` and `Destination` in events for lifecycle hooks, if that information is available\. For more information, see [Warm pool event types and patterns](warm-pools-eventbridge-events.md)\.

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

In this example event, Amazon EC2 Auto Scaling successfully terminated an instance\.

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

In this example event, Amazon EC2 Auto Scaling failed to terminate an instance\.

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

### EC2 Auto Scaling Instance Refresh Checkpoint Reached<a name="instance-refresh-checkpoint-reached"></a>

In this example event, the number of instances that have been replaced reaches the percentage threshold defined for the checkpoint\. 

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Checkpoint Reached",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "ab00cf8f-9126-4f3c-8010-dbb8cad6fb86",
      "AutoScalingGroupName": "my-auto-scaling-group",
      "CheckpointPercentage": "50",
      "CheckpointDelay": "300"
  }
}
```

### EC2 Auto Scaling Instance Refresh Started<a name="instance-refresh-started"></a>

In this example event, the status of an instance refresh changes to `InProgress`\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Started",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```

### EC2 Auto Scaling Instance Refresh Succeeded<a name="instance-refresh-succeeded"></a>

In this example event, the status of an instance refresh changes to `Succeeded`\. 

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Succeeded",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```

### EC2 Auto Scaling Instance Refresh Failed<a name="instance-refresh-failed"></a>

In this example event, the status of an instance refresh changes to `Failed`\. 

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Failed",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```

### EC2 Auto Scaling Instance Refresh Cancelled<a name="instance-refresh-cancelled"></a>

In this example event, the status of an instance refresh changes to `Cancelled`\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Auto Scaling Instance Refresh Cancelled",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": {
      "InstanceRefreshId": "c613620e-07e2-4ed2-a9e2-ef8258911ade",
      "AutoScalingGroupName": "my-auto-scaling-group"
  }
}
```