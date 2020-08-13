# Manual scaling for Amazon EC2 Auto Scaling<a name="as-manual-scaling"></a>

At any time, you can change the size of an existing Auto Scaling group manually\. You can either update the desired capacity of the Auto Scaling group, or update the instances that are attached to the Auto Scaling group\. Manually scaling your group can be useful when automatic scaling is not needed or when you need to hold capacity at a fixed number of instances\.

## Changing the size of your Auto Scaling group \(console\)<a name="as-manual-scaling-console"></a>

When you change the desired capacity of your Auto Scaling group, Amazon EC2 Auto Scaling manages the process of launching or terminating instances to maintain the new group size\.

The following example assumes that you've created an Auto Scaling group with a minimum size of 1 and a maximum size of 5\. Therefore, the group currently has one running instance\.

**To change the size of your Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Group details**, **Edit**\. \(Old console: On the **Details** tab, choose **Edit**\.\)

1. For **Desired capacity**, increase the desired capacity by one\. For example, if the current value is `1`, enter `2`\.

   The desired capacity must be less than or equal to the maximum size of the group\. If your new value for **Desired capacity** is greater than **Maximum capacity**, you must update **Maximum capacity**\.

1. When you are finished, choose **Update**\.

Now, verify that your Auto Scaling group has launched one additional instance\.

**To verify that the size of your Auto Scaling group has changed**

1. On the **Activity** tab, in **Activity history**, the **Status** column shows the current status of your instance\. Use the refresh button until you see the status of your instance change to **Successful**\. This indicates that your Auto Scaling group has successfully launched a new instance\. 
**Note**  
If the instance fails to launch, you can find troubleshooting tips in [Troubleshooting Amazon EC2 Auto Scaling](CHAP_Troubleshooting.md)\.

1. On the **Instance management** tab, in **Instances**, the **Lifecycle** column shows the state of your instances\. It takes a short time for an instance to launch\. After the instance starts, its state changes to `InService`\. You can see that your Auto Scaling group has launched `1` new instance, and it is in the `InService` state\.

## Changing the size of your Auto Scaling group \(AWS CLI\)<a name="as-manual-scaling-aws-cli"></a>

When you change the size of your Auto Scaling group, Amazon EC2 Auto Scaling manages the process of launching or terminating instances to maintain the new group size\. The default behavior is not to wait for the default cooldown period to complete, but you can override the default and wait for the cooldown period to complete\. For more information, see [Scaling cooldowns for Amazon EC2 Auto Scaling](Cooldown.md)\.

The following example assumes that you've created an Auto Scaling group with a minimum size of 1 and a maximum size of 5\. Therefore, the group currently has one running instance\.

**To change the size of your Auto Scaling group**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html) command to change the size of your Auto Scaling group, as shown in the following example\.

```
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg \
  --desired-capacity 2
```

If you choose to honor the default cooldown period for your Auto Scaling group, you must specify the `–-honor-cooldown` option as shown in the following example\.

```
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg \
  --desired-capacity 2 --honor-cooldown
```

**To verify the size of your Auto Scaling group**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to confirm that the size of your Auto Scaling group has changed, as in the following example\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
```

The following is example output, with details about the group and instances launched\.

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

Notice that `DesiredCapacity` shows the new value\. Your Auto Scaling group has launched an additional instance\.