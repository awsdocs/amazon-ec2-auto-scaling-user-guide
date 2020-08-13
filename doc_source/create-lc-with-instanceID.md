# Creating a launch configuration using an EC2 instance<a name="create-lc-with-instanceID"></a>

Amazon EC2 Auto Scaling provides you with an option to create a launch configuration using the attributes from a running EC2 instance\.

If the specified instance has properties that are not currently supported by launch configurations, the instances launched by the Auto Scaling group might not be identical to the original EC2 instance\.

There are differences between creating a launch configuration from scratch and creating a launch configuration from an existing EC2 instance\. When you create a launch configuration from scratch, you specify the image ID, instance type, optional resources \(such as storage devices\), and optional settings \(like monitoring\)\. When you create a launch configuration from a running instance, Amazon EC2 Auto Scaling derives attributes for the launch configuration from the specified instance\. Attributes are also derived from the block device mapping for the AMI from which the instance was launched, ignoring any additional block devices that were added after launch\.

When you create a launch configuration using a running instance, you can override the following attributes by specifying them as part of the same request: AMI, block devices, key pair, instance profile, instance type, kernel, instance monitoring, placement tenancy, ramdisk, security groups, Spot \(max\) price, user data, whether the instance has a public IP address is associated, and whether the instance is EBS\-optimized\.

**Tip**  
You can [create an Auto Scaling group directly from an EC2 instance](create-asg-from-instance.md)\. When you use this feature, Amazon EC2 Auto Scaling automatically creates a launch configuration for you as well\.

The following examples show you to create a launch configuration from an EC2 instance\.

**Topics**
+ [Create a launch configuration using an EC2 instance](#create-lc-with-defaultconfig)
+ [Create a launch configuration from an instance and override the block devices \(AWS CLI\)](#create-lc-with-bdm)
+ [Create a launch configuration and override the instance type \(AWS CLI\)](#create-lc-with-instance-type)

## Create a launch configuration using an EC2 instance<a name="create-lc-with-defaultconfig"></a>

To create a launch configuration using the attributes of an existing EC2 instance, specify the ID of the instance\.

**Important**  
The AMI used to launch the specified instance must still exist\.

### Create a launch configuration from an EC2 instance \(console\)<a name="create-lc-from-instance-console"></a>

You can use the console to create a launch configuration and an Auto Scaling group from a running EC2 instance and add the instance to the new Auto Scaling group\. For more information, see [Attach EC2 instances to your Auto Scaling group](attach-instance-asg.md)\.

### Create a launch configuration from an EC2 instance \(AWS CLI\)<a name="create-lc-with-defaultconfig-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command to create a launch configuration from an instance using the same attributes as the instance\. Any block devices added after launch are ignored\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc-from-instance --instance-id i-a8e09d9c
```

You can use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to describe the launch configuration and verify that its attributes match those of the instance\.

```
aws autoscaling describe-launch-configurations --launch-configuration-names my-lc-from-instance
```

The following is an example response\.

```
{
    "LaunchConfigurations": [
        {
            "UserData": null,
            "EbsOptimized": false,
            "LaunchConfigurationARN": "arn",
            "InstanceMonitoring": {
                "Enabled": false
            },
            "ImageId": "ami-05355a6c",
            "CreatedTime": "2014-12-29T16:14:50.382Z",
            "BlockDeviceMappings": [],
            "KeyName": "my-key-pair",
            "SecurityGroups": [
                "sg-8422d1eb"
            ],
            "LaunchConfigurationName": "my-lc-from-instance",
            "KernelId": "null",
            "RamdiskId": null,
            "InstanceType": "t1.micro",
            "AssociatePublicIpAddress": true
        }
    ]
}
```

## Create a launch configuration from an instance and override the block devices \(AWS CLI\)<a name="create-lc-with-bdm"></a>

By default, Amazon EC2 Auto Scaling uses the attributes from the EC2 instance that you specify to create the launch configuration\. However, the block devices come from the AMI used to launch the instance, not the instance\. To add block devices to the launch configuration, override the block device mapping for the launch configuration\.

**Important**  
The AMI used to launch the specified instance must still exist\.

### Create a launch configuration and override the block devices<a name="create-lc-with-bdm-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command to create a launch configuration using an EC2 instance but with a custom block device mapping\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc-from-instance-bdm --instance-id i-a8e09d9c \
  --block-device-mappings "[{\"DeviceName\":\"/dev/sda1\",\"Ebs\":{\"SnapshotId\":\"snap-3decf207\"}},{\"DeviceName\":\"/dev/sdf\",\"Ebs\":{\"SnapshotId\":\"snap-eed6ac86\"}}]"
```

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to describe the launch configuration and verify that it uses your custom block device mapping\.

```
aws autoscaling describe-launch-configurations --launch-configuration-names my-lc-from-instance-bdm
```

The following example response describes the launch configuration\.

```
{
    "LaunchConfigurations": [
        {
            "UserData": null,
            "EbsOptimized": false,
            "LaunchConfigurationARN": "arn",
            "InstanceMonitoring": {
                "Enabled": false
            },
            "ImageId": "ami-c49c0dac",
            "CreatedTime": "2015-01-07T14:51:26.065Z",
            "BlockDeviceMappings": [
                {
                    "DeviceName": "/dev/sda1",
                    "Ebs": {
                        "SnapshotId": "snap-3decf207"
                    }
                },
                {
                    "DeviceName": "/dev/sdf",
                    "Ebs": {
                        "SnapshotId": "snap-eed6ac86"
                    }
                }
            ],
            "KeyName": "my-key-pair",
            "SecurityGroups": [
                "sg-8637d3e3"
            ],
            "LaunchConfigurationName": "my-lc-from-instance-bdm",
            "KernelId": null,
            "RamdiskId": null,
            "InstanceType": "t1.micro",
            "AssociatePublicIpAddress": true
        }
    ]
}
```

## Create a launch configuration and override the instance type \(AWS CLI\)<a name="create-lc-with-instance-type"></a>

By default, Amazon EC2 Auto Scaling uses the attributes from the EC2 instance you specify to create the launch configuration\. Depending on your requirements, you might want to override attributes from the instance and use the values that you need\. For example, you can override the instance type\.

**Important**  
The AMI used to launch the specified instance must still exist\.

### Create a launch configuration and override the instance type<a name="create-lc-with-instance-type-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command to create a launch configuration using an EC2 instance but with a different instance type \(for example `t2.medium`\) than the instance \(for example `t2.micro`\)\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc-from-instance-changetype \
  --instance-id i-a8e09d9c --instance-type t2.medium
```

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to describe the launch configuration and verify that the instance type was overridden\.

```
aws autoscaling describe-launch-configurations --launch-configuration-names my-lc-from-instance-changetype
```

The following example response describes the launch configuration\.

```
{
    "LaunchConfigurations": [
        {
            "UserData": null,
            "EbsOptimized": false,
            "LaunchConfigurationARN": "arn",
            "InstanceMonitoring": {
                "Enabled": false
            },
            "ImageId": "ami-05355a6c",
            "CreatedTime": "2014-12-29T16:14:50.382Z",
            "BlockDeviceMappings": [],
            "KeyName": "my-key-pair",
            "SecurityGroups": [
                "sg-8422d1eb"
            ],
            "LaunchConfigurationName": "my-lc-from-instance-changetype",
            "KernelId": "null",
            "RamdiskId": null,
            "InstanceType": "t2.medium",
            "AssociatePublicIpAddress": true
        }
    ]
}
```