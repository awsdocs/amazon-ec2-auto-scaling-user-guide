# Warm pool event types and patterns<a name="warm-pools-eventbridge-events"></a>

**Note**  
Amazon EC2 Auto Scaling supports several predefined patterns in Amazon EventBridge\. This simplifies how an event pattern is created\. You select field values on a form, and EventBridge generates the pattern for you\. At this time, Amazon EC2 Auto Scaling doesn't support predefined patterns for any events that are emitted by an Auto Scaling group with a warm pool\. You must enter the pattern as a JSON object\. This section and the [Create EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md) topic show you how to use an event pattern to select events and send them to targets\. 

When you add lifecycle hooks to your Auto Scaling group, events are sent to EventBridge when an instance transitions into a wait state\.

There are two primary event types for lifecycle hooks: 
+ `EC2 Instance-launch Lifecycle Action`
+ `EC2 Instance-terminate Lifecycle Action`

To create EventBridge rules that filter for specific events that are emitted by an Auto Scaling group with a warm pool, include the `Origin` and `Destination` fields from the `detail` section of the event\. Events that contain the `Origin` and `Destination` fields are only emitted by Auto Scaling groups that have a warm pool\. 

The values of `Origin` and `Destination` can be the following:

`EC2` \| `AutoScalingGroup` \| `WarmPool`

**Topics**
+ [Example events](#warm-pool-event-types)
+ [Example event patterns](#warm-pools-eventbridge-patterns)

## Example events<a name="warm-pool-event-types"></a>

This section lists example events that are sent to EventBridge when an instance transitions into a wait state\. Events are emitted on a best\-effort basis\.

**Topics**
+ [Example 1: Amazon EC2 Auto Scaling adds a new instance to the warm pool](#warm-pool-example-1)
+ [Example 2: Amazon EC2 Auto Scaling adds an instance to the Auto Scaling group](#warm-pool-example-2)
+ [Example 3: Amazon EC2 Auto Scaling adds a new instance to the Auto Scaling group](#warm-pool-example-3)
+ [Example 4: Amazon EC2 Auto Scaling returns an instance to the warm pool](#warm-pool-example-4)

### Example 1: Amazon EC2 Auto Scaling adds a new instance to the warm pool<a name="warm-pool-example-1"></a>

In this example event, the state of a new instance changes to `Warmed:Pending:Wait` when it is added to a warm pool\. This occurs because of a lifecycle hook for scale\-out events\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
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
    "LifecycleHookName": "my-launch-lifecycle-hook", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info",
    "Origin": "EC2",
    "Destination": "WarmPool"
  } 
}
```

### Example 2: Amazon EC2 Auto Scaling adds an instance to the Auto Scaling group<a name="warm-pool-example-2"></a>

In this example event, the state of an instance in the warm pool changes to `Pending:Wait` when it is added to the Auto Scaling group\. This occurs because of a lifecycle hook for scale\-out events\.

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
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
    "LifecycleHookName": "my-launch-lifecycle-hook", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info",
    "Origin": "WarmPool",
    "Destination": "AutoScalingGroup"
  } 
}
```

### Example 3: Amazon EC2 Auto Scaling adds a new instance to the Auto Scaling group<a name="warm-pool-example-3"></a>

In this example event, the state of a new instance \(not an instance from the warm pool\) changes to `Pending:Wait` when it is added to the Auto Scaling group\. This occurs because of a lifecycle hook for scale\-out events\.

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
    "LifecycleHookName": "my-launch-lifecycle-hook", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_LAUNCHING",
    "NotificationMetadata": "additional-info",
    "Origin": "EC2",
    "Destination": "AutoScalingGroup"
  } 
}
```

### Example 4: Amazon EC2 Auto Scaling returns an instance to the warm pool<a name="warm-pool-example-4"></a>

In this example event, the state of an instance changes to `Warmed:Pending:Wait` when it is returned to the warm pool\. This occurs because of a lifecycle hook for scale\-in events\. 

```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance-terminate Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "2022-03-28T00:12:37.214Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:042cba90-ad2f-431c-9b4d-6d9055bcc9fb:autoScalingGroupName/my-asg"
  ],
  "detail": { 
    "LifecycleActionToken": "42694b3d-4b70-6a62-8523-09a1eEXAMPLE", 
    "AutoScalingGroupName": "my-asg", 
    "LifecycleHookName": "my-termination-lifecycle-hook", 
    "EC2InstanceId": "i-1234567890abcdef0", 
    "LifecycleTransition": "autoscaling:EC2_INSTANCE_TERMINATING",
    "NotificationMetadata": "additional-info",
    "Origin": "AutoScalingGroup",
    "Destination": "WarmPool"
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
  "source": [ "aws.autoscaling" ],
  "detail-type": [ "EC2 Instance-launch Lifecycle Action" ],
  "detail": {
      "Origin": [ "EC2" ],
      "Destination": [ "WarmPool" ]
   }
}
```

Use the following sample event pattern to capture all events that are associated with instances leaving the warm pool because of a scale\-out event\.

```
{
  "source": [ "aws.autoscaling" ],
  "detail-type": [ "EC2 Instance-launch Lifecycle Action" ],
  "detail": {
      "Origin": [ "WarmPool" ],
      "Destination": [ "AutoScalingGroup" ]
   }
}
```

Use the following sample event pattern to capture all events that are associated with instances launching directly into the Auto Scaling group\.

```
{
  "source": [ "aws.autoscaling" ],
  "detail-type": [ "EC2 Instance-launch Lifecycle Action" ],
  "detail": {
      "Origin": [ "EC2" ],
      "Destination": [ "AutoScalingGroup" ]
   }
}
```

Use the following sample event pattern to capture all events that are associated with instances returning to the warm pool on scale in\.

```
{
  "source": [ "aws.autoscaling" ],
  "detail-type": [ "EC2 Instance-terminate Lifecycle Action" ],
  "detail": {
      "Origin": [ "AutoScalingGroup" ],
      "Destination": [ "WarmPool" ]
   }
}
```

Use the following sample event pattern to capture all events that are associated with `EC2 Instance-launch Lifecycle Action`, regardless of the origin or destination\.

```
{
  "source": [ "aws.autoscaling" ],
  "detail-type": [ "EC2 Instance-launch Lifecycle Action" ]
}
```