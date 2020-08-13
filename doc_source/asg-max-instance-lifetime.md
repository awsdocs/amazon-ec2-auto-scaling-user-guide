# Replacing Auto Scaling instances based on maximum instance lifetime<a name="asg-max-instance-lifetime"></a>

When you use the AWS Management Console to update an Auto Scaling group, or when you use the AWS CLI or AWS SDKs to create or update an Auto Scaling group, you can set the optional maximum instance lifetime parameter\. The maximum instance lifetime feature does the work of replacing instances that have been in service for the maximum amount of time allowed\. For example, this feature supports common compliance use cases, such as being required to replace your instances on a schedule due to internal security policies or external compliance controls\. This topic describes the key aspects of this feature and how to configure it for your Auto Scaling group\.

The maximum instance lifetime specifies the maximum amount of time \(in seconds\) that an instance can be in service\. The maximum duration applies to all current and future instances in the group\. As an instance approaches its maximum duration, it is terminated and replaced, and cannot be used again\.

When configuring the maximum instance lifetime for your Auto Scaling group, you must specify a value of at least 604,800 seconds \(7 days\)\. To clear a previously set value, specify a new value of 0\.

Note that instances are not guaranteed to be replaced only at the end of their maximum duration\. In some situations, Amazon EC2 Auto Scaling might need to start replacing instances immediately after you configure the maximum instance lifetime parameter\. The intention of this more aggressive behavior is to avoid replacing all instances at the same time\. 

Depending on the maximum duration specified and the size of the Auto Scaling group, the rate of replacement may vary\. In general, Amazon EC2 Auto Scaling replaces instances one at a time, with a pause in between replacements\. However, the rate of replacement will be higher when there is not enough time to replace each instance individually based on the maximum duration that you specified\. In this case, Amazon EC2 Auto Scaling will replace several instances at once, by up to 10 percent of the current capacity of your Auto Scaling group at a time\.

To manage the rate of replacement, you can do the following:
+ Set the maximum instance lifetime limit to a longer period of time to space out the replacements\. This is helpful for groups that have a large number of instances to replace\.
+ Add extra time between certain replacements by using instance protection to temporarily prevent individual instances in your Auto Scaling group from being replaced\. When you're ready to replace these instances, remove instance protection from each individual instance\. For more information, see [Instance scale\-in protection](as-instance-termination.md#instance-protection)\.

**To configure maximum instance lifetime \(console\)**  
Create the Auto Scaling group in the usual way\. After creating the Auto Scaling group, edit the group to specify the maximum instance lifetime\. 

**To configure maximum instance lifetime \(AWS CLI\)**  
When specifying the maximum instance lifetime using the AWS CLI, you can apply this limit to an existing Auto Scaling group\. You can also apply this limit to a new Auto Scaling group as you create it\. 

For new Auto Scaling groups, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command\.

```
aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
```

The following is an example `config.json` file that shows a maximum instance lifetime of `2592000` seconds \(30 days\)\.

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