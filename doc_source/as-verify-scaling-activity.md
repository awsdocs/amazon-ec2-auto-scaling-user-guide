# Verifying a scaling activity for an Auto Scaling group<a name="as-verify-scaling-activity"></a>

In the AWS Management Console, the **Activity history** for an Auto Scaling group lets you view the current status of a scaling activity that is currently in progress\. When the scaling activity is finished, you can see whether it succeeds or not\. This is particularly useful when you are creating Auto Scaling groups or you are adding scaling conditions to existing groups\.

When you add a target tracking, step, or simple scaling policy to your Auto Scaling group, Amazon EC2 Auto Scaling immediately starts evaluating the policy against the metric\. The metric alarm goes to ALARM state when the metric breaches the threshold for a specified number of evaluation periods\. This means that a scaling policy could result in a scaling activity soon after it's created\. After Amazon EC2 Auto Scaling adjusts the desired capacity in response to a scaling policy, you can verify the scaling activity in your account\. If you want to receive email notification from Amazon EC2 Auto Scaling informing you about a scaling activity, follow the instructions in [Getting Amazon SNS notifications when your Auto Scaling group scales](ASGettingNotifications.md)\.

**To view the scaling activities for an Auto Scaling group \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched or terminated instances, or whether the scaling activity is still in progress\.

   If you have a lot of scaling activities, you can choose the **>** icon at the top edge of the activity history to see the next page of scaling activities\. You can also choose the gear icon and increase the amount of results shown per page\.

1. On the **Instance management** tab, under **Instances**, the **Lifecycle** column contains the state of your instances\. After the instance starts and any lifecycle hooks have finished, its lifecycle state changes to `InService`\.

   The **Health status** column shows the result of the EC2 instance health check on your instance\.

**To view the scaling activities for an Auto Scaling group \(AWS CLI\)**  
Use the following [describe\-scaling\-activities](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command\. 

```
aws autoscaling describe-scaling-activities --auto-scaling-group-name my-asg
```

The following is example output\. 

Scaling activities are ordered by start time\. Activities still in progress are described first\. 

```
{
  "Activities": [
    {
      "ActivityId": "5e3a1f47-2309-415c-bfd8-35aa06300799",
      "AutoScalingGroupName": "my-asg",
      "Description": "Terminating EC2 instance: i-06c4794c2499af1df",
      "Cause": "At 2020-02-11T18:34:10Z a monitor alarm TargetTracking-my-asg-AlarmLow-b9376cab-18a7-4385-920c-dfa3f7783f82 in state ALARM triggered policy my-target-tracking-policy changing the desired capacity from 3 to 2.  At 2020-02-11T18:34:31Z an instance was taken out of service in response to a difference between desired and actual capacity, shrinking the capacity from 3 to 2.  At 2020-02-11T18:34:31Z instance i-06c4794c2499af1df was selected for termination.",
      "StartTime": "2020-02-11T18:34:31.268Z",
      "EndTime": "2020-02-11T18:34:53Z",
      "StatusCode": "Successful",
      "Progress": 100,
      "Details": "{\"Subnet ID\":\"subnet-5ea0c127\",\"Availability Zone\":\"us-west-2a\"...}",
      "AutoScalingGroupARN": "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:283179a2-f3ce-423d-93f6-66bb518232f7:autoScalingGroupName/my-asg"
    },
...
  ]
}
```

For information about the types of errors that you may encounter and how to handle them, see [Troubleshooting Amazon EC2 Auto Scaling](CHAP_Troubleshooting.md)\.