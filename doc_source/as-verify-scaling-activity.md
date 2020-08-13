# Verifying a scaling activity for an Auto Scaling group<a name="as-verify-scaling-activity"></a>

After you create a scaling policy, Amazon EC2 Auto Scaling starts evaluating the policy against the metric\. The metric alarm goes to ALARM state when the metric breaches the threshold for a specified number of evaluation periods\. This means that a scaling policy could trigger a scaling action soon after it's created\. After Amazon EC2 Auto Scaling changes capacity in response to a scaling policy, you can verify the scaling activity in your account\. If you want to receive email notification from Amazon EC2 Auto Scaling informing you that a scaling action was triggered, follow the instructions in [Getting Amazon SNS notifications when your Auto Scaling group scales](ASGettingNotifications.md)\.

**To view the scaling activities for your Auto Scaling group \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. The original console is open by default\. To access the new console, on the banner at the top of the page, choose **Go to the new console**\.

1. Search for the name of the Auto Scaling group\.

   The **Instances** column shows the number of instances currently running\. While an instance is being launched or terminated, the **Status** column displays a status of "Updating capacity\." Wait for a few minutes, and then refresh the view to see the latest status\. After a scaling activity completes, notice that the **Instances** and **Desired capacity** columns show new values\. 
**Note**  
If you're using instance weighting, the **Weighted capacity** column measures the number of capacity units that the group contains\. If this column is hidden, choose the gear\-shaped icon in the top\-right corner of the section and then enable **Weighted capacity**\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

   1. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched or terminated instances\.

   1. On the **Instance management** tab, under **Instances**, you can view the status of the instances that are currently running\. The **Lifecycle** column contains the state of your instances\. Note that it takes a short time for an instance to launch\. After the instance starts, its lifecycle state changes to `InService`\.

**To view the scaling activities for your Auto Scaling group \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command\. 

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
      "Details": "{\"Subnet ID\":\"subnet-5ea0c127\",\"Availability Zone\":\"us-west-2a\"...}"
    },
...
  ]
}
```

**To verify the size of your Auto Scaling group \(AWS CLI\)**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
```

The following is example output, with details about the group and the currently running instances\. 

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupARN": "arn",
            "ServiceLinkedRoleARN": "arn",
            "TargetGroupARNs": [],
            "SuspendedProcesses": [],
            "LaunchTemplate": {
                "LaunchTemplateName": "my-launch-template",
                "Version": "1",
                "LaunchTemplateId": "lt-050555ad16a3f9c7f"
            },
            "Tags": [],
            "EnabledMetrics": [],
            "LoadBalancerNames": [],
            "AutoScalingGroupName": "my-asg",
            "DefaultCooldown": 300,
            "MinSize": 1,
            "Instances": [
                {
                    "ProtectedFromScaleIn": false,
                    "AvailabilityZone": "us-west-2a",
                    "LaunchTemplate": {
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "1",
                        "LaunchTemplateId": "lt-050555ad16a3f9c7f"
                    },
                    "InstanceId": "i-05b4f7d5be44822a6",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "Pending"
                },
                {
                    "ProtectedFromScaleIn": false,
                    "AvailabilityZone": "us-west-2a",
                    "LaunchTemplate": {
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "1",
                        "LaunchTemplateId": "lt-050555ad16a3f9c7f"
                    },
                    "InstanceId": "i-0c20ac468fa3049e8",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService"
                }
            ],
            "MaxSize": 5,
            "VPCZoneIdentifier": "subnet-c87f2be0",
            "HealthCheckGracePeriod": 300,
            "TerminationPolicies": [
                "Default"
            ],
            "CreatedTime": "2019-03-18T23:30:42.611Z",
            "AvailabilityZones": [
                "us-west-2a"
            ],
            "HealthCheckType": "EC2",
            "NewInstancesProtectedFromScaleIn": false,
            "DesiredCapacity": 2
        }
    ]
}
```