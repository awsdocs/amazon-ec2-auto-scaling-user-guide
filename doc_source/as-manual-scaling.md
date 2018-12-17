# Manual Scaling<a name="as-manual-scaling"></a>

At any time, you can change the size of an existing Auto Scaling group\. Update the desired capacity of the Auto Scaling group, or update the instances that are attached to the Auto Scaling group\.

**Topics**
+ [Change the Size of Your Auto Scaling Group Using the Console](#as-manual-scaling-console)
+ [Change the Size of Your Auto Scaling Group Using the AWS CLI](#as-manual-scaling-aws-cli)
+ [Attach EC2 Instances to Your Auto Scaling Group](attach-instance-asg.md)
+ [Detach EC2 Instances from Your Auto Scaling Group](detach-instance-asg.md)

## Change the Size of Your Auto Scaling Group Using the Console<a name="as-manual-scaling-console"></a>

When you change the size of your Auto Scaling group, Amazon EC2 Auto Scaling manages the process of launching or terminating instances to maintain the new group size\.

The following example assumes that you've created an Auto Scaling group with a minimum size of 1 and a maximum size of 5\. Therefore, the group currently has one running instance\.

**To change the size of your Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Details** tab, choose **Edit**\.

1. For **Desired**, increase the desired capacity by one\. For example, if the current value is `1`, type `2`\.

   The desired capacity must be less than or equal to the maximum size of the group\. If your new value for **Desired** is greater than **Max**, you must update **Max** \.

   When you are finished, choose **Save**\.

Now, verify that your Auto Scaling group has launched one additional instance\.

**To verify that the size of your Auto Scaling group has changed**

1. On the **Activity History** tab, the **Status** column shows the current status of your instance\. You can use the refresh button until you see the status of your instance change to **Successful**, indicating that your Auto Scaling group has successfully launched a new instance\.

1. On the **Instances** tab, the **Lifecycle** column shows the state of your instances\. It takes a short time for an instance to launch\. After the instance starts, its state changes to `InService`\. You can see that your Auto Scaling group has launched `1` new instance, and it is in the `InService` state\.

## Change the Size of Your Auto Scaling Group Using the AWS CLI<a name="as-manual-scaling-aws-cli"></a>

When you change the size of your Auto Scaling group, Amazon EC2 Auto Scaling manages the process of launching or terminating instances to maintain the new group size\.

The following example assumes that you've created an Auto Scaling group with a minimum size of 1 and a maximum size of 5\. Therefore, the group currently has one running instance\.

Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html) command to change the size of your Auto Scaling group, as shown in the following example:

```
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg --desired-capacity 2
```

By default, the command does not wait for the cooldown period specified for the group to complete\. You can override the default behavior and wait for the cooldown period to complete by specifying the `–-honor-cooldown` option as shown in the following example\. For more information, see [Scaling Cooldowns for Amazon EC2 Auto Scaling](Cooldown.md)\.

```
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg --desired-capacity 2 --honor-cooldown
```

Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to confirm that the size of your Auto Scaling group has changed, as in the following example:

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
```

The following is example output, with details about the group and instances launched\.

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupARN": "arn",
            "HealthCheckGracePeriod": 300,
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
                    "InstanceId": "i-33388a3f",
                    "AvailabilityZone": "us-west-2a",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService",
                    "LaunchConfigurationName": "my-lc"
                }
            ],
            "MaxSize": 5,
            "VPCZoneIdentifier": "subnet-e4f33493",
            "TerminationPolicies": [
                "Default"
            ],
            "LaunchConfigurationName": "my-lc",
            "CreatedTime": "2014-12-12T23:30:42.611Z",
            "AvailabilityZones": [
                "us-west-2a"
            ],
            "HealthCheckType": "EC2"
        }
    ]
}
```

Notice that `DesiredCapacity` shows the new value\. Your Auto Scaling group has launched an additional instance\.