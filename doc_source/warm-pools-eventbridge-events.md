# Event types and event patterns that you use when you add or update lifecycle hooks<a name="warm-pools-eventbridge-events"></a>

When you add lifecycle hooks to your Auto Scaling group, events are sent to EventBridge in JSON format\. You can create an EventBridge rule that uses an event pattern to filter incoming events and then invokes your Lambda function or other target\.

There are two types of events that are emitted for lifecycle hooks: 
+ `EC2 Instance-launch Lifecycle Action`
+ `EC2 Instance-terminate Lifecycle Action`\. 

After you add a warm pool to your Auto Scaling group, the `detail` section for `EC2 Instance-launch Lifecycle Action` events contains new `Origin` and `Destination` fields\. 

The values of `Origin` and `Destination` can be the following:

`EC2` \| `AutoScalingGroup` \| `WarmPool` 

**Topics**
+ [Warm pool events](#warm-pool-event-types)
+ [Example event patterns](#warm-pools-eventbridge-patterns)

## Warm pool events<a name="warm-pool-event-types"></a>

This section lists example events from Amazon EC2 Auto Scaling\. Events are emitted on a best\-effort basis\.

**Instances bound for the warm pool**  
The following example shows an event for a lifecycle hook for the `Warmed:Pending:Wait` state\. It represents an instance that is launching into the warm pool\. 

```
{
  "version": "0",
  "id": "18b2ec17-3e9b-4c15-8024-ff2e8ce8786a",
  "detail-type": "EC2 Instance-launch Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "2021-01-13T00:12:37.214Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:042cba90-ad2f-431c-9b4d-6d9055bcc9fb:autoScalingGroupName/my-asg"
  ],
  "detail": { 
    "LifecycleActionToken": "71514b9d-6a40-4b26-8523-05e7eEXAMPLE", 
    "AutoScalingGroupName": "my-asg", 
    "LifecycleHookName": "my-lifecycle-hook-for-warming-instances", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info",
    "Origin": "EC2",
    "Destination": "WarmPool"
  } 
}
```

**Instances leaving the warm pool**  
The following example shows an event for a lifecycle hook for the `Pending:Wait` state\. It represents an instance that is leaving the warm pool due to a scale\-out event\.

```
{
  "version": "0",
  "id": "5985cdde-1f01-9ae2-1498-3c8ea162d141",
  "detail-type": "EC2 Instance-launch Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "2021-01-19T00:35:52.359Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:042cba90-ad2f-431c-9b4d-6d9055bcc9fb:autoScalingGroupName/my-asg"
  ],
  "detail": { 
    "LifecycleActionToken": "19cc4d4a-e450-4d1c-b448-0de67EXAMPLE", 
    "AutoScalingGroupName": "my-asg", 
    "LifecycleHookName": "my-lifecycle-hook-for-warmed-instances", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info",
    "Origin": "WarmPool",
    "Destination": "AutoScalingGroup"
  } 
}
```

**Instances launching outside of the warm pool**  
The following example shows an event for a lifecycle hook for the `Pending:Wait` state\. It represents an instance that is launching directly into the Auto Scaling group\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance-launch Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "2021-02-01T17:18:06.082Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:042cba90-ad2f-431c-9b4d-6d9055bcc9fb:autoScalingGroupName/my-asg"
  ],
  "detail": { 
    "LifecycleActionToken": "87654321-4321-4321-4321-21098EXAMPLE", 
    "AutoScalingGroupName": "my-asg", 
    "LifecycleHookName": "my-lifecycle-hook-for-launching-instances", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info",
    "Origin": "EC2",
    "Destination": "AutoScalingGroup"
  } 
}
```

## Example event patterns<a name="warm-pools-eventbridge-patterns"></a>

The preceding section provides example events emitted by Amazon EC2 Auto Scaling\.

EventBridge event patterns have the same structure as the events that they match\. The pattern quotes the fields that you want to match and provides the values that you're looking for\. 

The following fields in the event form the event pattern that is defined in the rule to invoke an action:

`"source": "aws.autoscaling"`  
Identifies that the event is from Amazon EC2 Auto Scaling\.

`"detail-type": "EC2 Instance-launch Lifecycle Action"`  
Identifies the event type\. 

`"Origin": "EC2"`  
Identifies where the instance is coming from\. 

`"Destination": "WarmPool"`  
Identifies where the instance is going to\. 

Use the following sample event pattern to capture all events that are associated with instances entering the warm pool\.

```
{
  "source": [ 
      "aws.autoscaling" 
  ],
  "detail-type": [ 
      "EC2 Instance-launch Lifecycle Action" 
  ],
  "detail": {
      "Origin": [
          "EC2"
      ],
      "Destination": [
          "WarmPool"
      ]
   }
}
```

Use the following sample event pattern to capture all events that are associated with instances leaving the warm pool due to a scale\-out event\.

```
{
  "source": [ 
      "aws.autoscaling" 
  ],
  "detail-type": [ 
      "EC2 Instance-launch Lifecycle Action" 
  ],
  "detail": {
      "Origin": [
          "WarmPool"
      ],
      "Destination": [
          "AutoScalingGroup"
      ]
   }
}
```

Use the following sample event pattern to capture all events that are associated with instances launching directly into the Auto Scaling group\.

```
{
  "source": [ 
      "aws.autoscaling" 
  ],
  "detail-type": [ 
      "EC2 Instance-launch Lifecycle Action" 
  ],
  "detail": {
      "Origin": [
          "EC2"
      ],
      "Destination": [
          "AutoScalingGroup"
      ]
   }
}
```

Use the following sample event pattern to capture all events that are associated with `EC2 Instance-launch Lifecycle Action`, regardless of the origin or destination\.

```
{
  "source": [ 
      "aws.autoscaling" 
  ],
  "detail-type": [ 
      "EC2 Instance-launch Lifecycle Action" 
  ]
}
```