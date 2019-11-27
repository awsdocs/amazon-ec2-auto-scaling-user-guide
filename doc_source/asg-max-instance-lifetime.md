# Replacing Auto Scaling Instances Based on Maximum Instance Lifetime<a name="asg-max-instance-lifetime"></a>

The maximum instance lifetime feature does the work of replacing instances that have been in service for the maximum amount of time allowed\. This topic describes the key aspects of this feature and how to configure it for your Auto Scaling group\.

To get started, configure a maximum instance lifetime limit for your Auto Scaling group\. This limit specifies the maximum amount of time \(in seconds\) that an instance can be in service\. The maximum duration applies to current and future instances in the group\. As an instance approaches its maximum duration, it is terminated by AWS, and cannot be used again\.

Note that instances are not guaranteed to be terminated at the end of their maximum duration\. In some situations, Amazon EC2 Auto Scaling might need to start replacing instances immediately after you configure the maximum instance lifetime limit\. The intention is to avoid replacing all instances at the same time\. 

Optionally, you can use instance protection if you do not want to replace specific instances in your Auto Scaling group\. For more information, see [Instance Scale\-In Protection](as-instance-termination.md#instance-protection)\.

**Important**  
Make sure that the length of time you specify is at least 604,800 seconds \(7 days\)\. This is the minimum requirement for maximum instance lifetime\. To clear a previously set value, specify a new value of 0\.

**To configure maximum instance lifetime \(console\)**  
Create the Auto Scaling group in the usual way\. After creating the Auto Scaling group, edit the group to specify the maximum instance lifetime\. 

**To configure maximum instance lifetime \(AWS CLI\)**  
When specifying the maximum instance lifetime using the AWS CLI, you can apply this limit to an existing Auto Scaling group\. You can also apply this limit to a new Auto Scaling group as you create it\. 

For new Auto Scaling groups, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command\.

```
aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
```

The following is an example `config.json` file\. 

```
{
    "AutoScalingGroupName": "my-asg",
    "LaunchTemplate": {
        "LaunchTemplateName": "my-launch-template",
        "Version": "$Latest"
    },
    "MinSize": 1,
    "MaxSize": 5,
    "MaxInstanceLifetime": 2592000,
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
    "Tags": []
}
```

For existing Auto Scaling groups, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-existing-asg --max-instance-lifetime 2592000
```

**To verify the maximum instance lifetime for an Auto Scaling group**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
```

The following is an example response\.

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupName": "my-asg",
            "AutoScalingGroupARN": "arn",
            "LaunchTemplate": {
                "LaunchTemplateId": "lt-0b97f1e282EXAMPLE",
                "LaunchTemplateName": "my-launch-template",
                "Version": "$Latest"
            },
            "MinSize": 1,
            "MaxSize": 5,
            "DesiredCapacity": 1,
            "DefaultCooldown": 300,
            "AvailabilityZones": [
                "us-west-2a",
                "us-west-2b",
                "us-west-2c"
            ],
            "LoadBalancerNames": [],
            "TargetGroupARNs": [],
            "HealthCheckType": "EC2",
            "HealthCheckGracePeriod": 0,
            "Instances": [
                {
                    "InstanceId": "i-04d180b9d5fc578fc",
                    "InstanceType": "t2.small",
                    "AvailabilityZone": "us-west-2b",
                    "LifecycleState": "Pending",
                    "HealthStatus": "Healthy",
                    "LaunchTemplate": {
                        "LaunchTemplateId": "lt-0b97f1e282EXAMPLE",
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "7"
                    },
                    "ProtectedFromScaleIn": false
                }
            ],
            "CreatedTime": "2019-11-14T22:56:15.487Z",
            "SuspendedProcesses": [],
            "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
            "EnabledMetrics": [],
            "Tags": [],
            "TerminationPolicies": [
                "Default"
            ],
            "NewInstancesProtectedFromScaleIn": false,
            "ServiceLinkedRoleARN": "arn",
            "MaxInstanceLifetime": 2592000
        }
    ]
}
```