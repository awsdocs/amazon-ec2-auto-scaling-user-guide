# Replace Auto Scaling instances based on maximum instance lifetime<a name="asg-max-instance-lifetime"></a>

The maximum instance lifetime specifies the maximum amount of time \(in seconds\) that an instance can be in service before it is terminated and replaced\. A common use case might be a requirement to replace your instances on a schedule because of internal security policies or external compliance controls\. 

You must specify a value of at least 86,400 seconds \(one day\)\. To clear a previously set value, specify a new value of 0\. This setting applies to all current and future instances in your Auto Scaling group\.

Setting this value too low can cause instances to be replaced faster than desired\. In general, Amazon EC2 Auto Scaling replaces instances one at a time, with a pause between replacements\. However, if the maximum instance lifetime that you specify doesn't provide enough time to replace each instance individually, Amazon EC2 Auto Scaling must replace more than one instance at a time\. Several instances might be replaced at once, by up to 10 percent of the current capacity of your Auto Scaling group\.

To manage the rate of replacement, you can do the following:
+ Set the maximum instance lifetime limit to a longer period of time\. This spaces out the replacements, which is helpful for groups that have a large number of instances to replace\.
+ Add extra time between certain replacements by using instance protection\. This temporarily prevents individual instances in your Auto Scaling group from being replaced\. When you're ready to replace these instances, remove instance protection from each individual instance\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

**Note**  
Whenever an old instance is replaced and a new instance launches, the new instance uses the launch template or launch configuration that is currently associated with the Auto Scaling group\. If your launch template or launch configuration specifies the AMI ID of a different version of your application, this version of your application will be deployed automatically\.

## Set the maximum instance lifetime<a name="set-maximum-instance-lifetime"></a>

When you create an Auto Scaling group in the console, you cannot set the maximum instance lifetime\. However, after the group is created, you can edit it to set the maximum instance lifetime\.

**To set the maximum instance lifetime for a group \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page, showing information about the group you selected\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\.

1. For **Maximum instance lifetime**, enter the maximum number of seconds that an instance can be in service\.

1. Choose **Update**\.

On the **Activity** tab, under **Activity history**, you can view the replacement of instances in the group throughout its history\. 

**To set the maximum instance lifetime for a group \(AWS CLI\)**  
You can also use the AWS CLI to set the maximum instance lifetime for new or existing Auto Scaling groups\.

For new Auto Scaling groups, use the [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command\.

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

For existing Auto Scaling groups, use the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-existing-asg --max-instance-lifetime 2592000
```

**To verify the maximum instance lifetime for an Auto Scaling group**  
Use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

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

## Limitations<a name="maximum-instance-lifetime-limitations"></a>
+ **Maximum lifetime not guaranteed to be exact for every instance**: Instances are not guaranteed to be replaced only at the end of their maximum duration\. In some situations, Amazon EC2 Auto Scaling might need to start replacing instances immediately after you update the maximum instance lifetime parameter\. The reason for this behavior is to avoid replacing all instances at the same time\.
+ **Instances terminated before launch**: When there is only one instance in the Auto Scaling group, the maximum instance lifetime feature can result in an outage because Amazon EC2 Auto Scaling terminates an instance and then launches a new instance\.