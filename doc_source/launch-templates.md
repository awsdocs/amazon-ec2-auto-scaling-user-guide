# Launch templates<a name="launch-templates"></a>

A launch template is similar to a [launch configuration](launch-configurations.md), in that it specifies instance configuration information\. It includes the ID of the Amazon Machine Image \(AMI\), the instance type, a key pair, security groups, and other parameters used to launch EC2 instances\. However, defining a launch template instead of a launch configuration allows you to have multiple versions of a launch template\. 

With versioning of launch templates, you can create a subset of the full set of parameters\. Then, you can reuse it to create other versions of the same launch template\. For example, you can create a launch template that defines a base configuration without an AMI or user data script\. After you create your launch template, you can create a new version and add the AMI and user data that has the latest version of your application for testing\. This results in two versions of the launch template\. Storing a base configuration helps you to maintain the required general configuration parameters\. You can create a new version of your launch template from the base configuration whenever you want\. You can also delete the versions used for testing your application when you no longer need them\. 

We recommend that you use launch templates to ensure that you're accessing the latest features and improvements\. Not all Amazon EC2 Auto Scaling features are available when you use launch configurations\. For example, you cannot create an Auto Scaling group that launches both Spot and On\-Demand Instances or that specifies multiple instance types\. You must use a launch template to configure these features\. For more information, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.

With launch templates, you can also use newer features of Amazon EC2\. This includes Systems Manager parameters \(AMI ID\), the current generation of EBS Provisioned IOPS volumes \(io2\), EBS volume tagging, [T2 Unlimited instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-unlimited-mode-concepts.html), Elastic Inference, and [Dedicated Hosts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html), to name a few\. Dedicated Hosts are physical servers with EC2 instance capacity that are dedicated to your use\. While Amazon EC2 [Dedicated Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html) also run on dedicated hardware, the advantage of using Dedicated Hosts over Dedicated Instances is that you can bring eligible software licenses from external vendors and use them on EC2 instances\. 

When you create a launch template, all parameters are optional\. However, if a launch template does not specify an AMI, you cannot add the AMI when you create your Auto Scaling group\. If you specify an AMI but no instance type, you can add one or more instance types when you create your Auto Scaling group\.

**Topics**
+ [Permissions](#launch-templates-permissions)
+ [Migrate to launch templates](#migrate-to-launch-templates)
+ [Create a launch template for an Auto Scaling group](create-launch-template.md)
+ [Copy launch configurations to launch templates](copy-launch-config.md)
+ [Replace a launch configuration with a launch template](replace-launch-config.md)
+ [Request Spot Instances for fault\-tolerant and flexible applications](launch-template-spot-instances.md)
+ [Examples for creating and managing launch templates with the AWS Command Line Interface \(AWS CLI\)](examples-launch-templates-aws-cli.md)
+ [Using AWS Systems Manager parameters instead of AMI IDs in launch templates](using-systems-manager-parameters.md)

## Permissions<a name="launch-templates-permissions"></a>

The procedures in this section assume that you already have the required permissions to use launch templates\. With permissions in place, you can create and manage launch templates\. You can also create and update Auto Scaling groups and specify a launch template instead of a launch configuration\. 

When you update or create an Auto Scaling group and specify a launch template, your `ec2:RunInstances` permissions are checked\. If you do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. 

Some additional functionality in the request requires additional permissions, such as the ability to pass an IAM role to provisioned instances or to add tags to provisioned instances and volumes\.

For information about how an administrator grants you permissions, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

## Migrate to launch templates<a name="migrate-to-launch-templates"></a>

**Note**  
To migrate your AWS CloudFormation stacks from launch configurations to launch templates, see [Migrate AWS CloudFormation stacks from launch configurations to launch templates](migrate-launch-configurations-with-cloudformation.md)\.

A basic process for migrating to launch templates from launch configurations involves the following steps:

1. Migrate data from your existing launch configurations to launch templates by copying them in the console\. For more information, see [Copy launch configurations to launch templates](copy-launch-config.md)\.

1. Migrate each Auto Scaling group that uses a launch configuration to use a launch template by starting an instance refresh to replace existing instances with new instances\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\. 

   For very small Auto Scaling groups, you could replace the launch configurations with launch templates directly on the group\. For more information, see [Replace a launch configuration with a launch template](replace-launch-config.md)\. Then, to update the existing instances, you can terminate them directly from the Amazon EC2 console\. Manual termination forces your Auto Scaling group to launch new instances to maintain the group's desired capacity\.

For more information about migrating to launch templates, see [Amazon EC2 Auto Scaling will no longer add support for new EC2 features to Launch Configurations](http://aws.amazon.com/blogs/compute/amazon-ec2-auto-scaling-will-no-longer-add-support-for-new-ec2-features-to-launch-configurations/) on the AWS Compute Blog\.

### Find Auto Scaling groups that use launch configurations<a name="find-groups-that-use-launch-configurations"></a>

When migrating your Auto Scaling groups from launch configurations to launch templates, you might need to identify the Auto Scaling groups that are using launch configurations\. To do this, run the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\. Replace *REGION* with your AWS Region\.

```
aws autoscaling describe-auto-scaling-groups --region REGION \
  --query 'AutoScalingGroups[?LaunchConfigurationName!=`null`]'
```

For more information about filtering, see [Filtering AWS CLI output](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html) in the *AWS Command Line Interface User Guide*\.

The following is example output\.

```
[
    {
	"AutoScalingGroupName": "group-1",
	"AutoScalingGroupARN": "arn",
	"LaunchConfigurationName": "my-launch-config",
	"MinSize": 1,
	"MaxSize": 5,
	"DesiredCapacity": 2,
	"DefaultCooldown": 300,
	"AvailabilityZones": [
            "us-west-2a",
            "us-west-2b",
            "us-west-2c"
        ],
	"LoadBalancerNames": [],
	"TargetGroupARNs": [],
	"HealthCheckType": "EC2",
	"HealthCheckGracePeriod": 300,
	"Instances": [
            {
                "ProtectedFromScaleIn": false,
                "AvailabilityZone": "us-west-2a",
                "LaunchConfigurationName": "my-launch-config",
                "InstanceId": "i-05b4f7d5be44822a6",
                "InstanceType": "t3.micro",
                "HealthStatus": "Healthy",
                "LifecycleState": "InService"
            },
            {
                "ProtectedFromScaleIn": false,
                "AvailabilityZone": "us-west-2b",
                "LaunchConfigurationName": "my-launch-config",
                "InstanceId": "i-0c20ac468fa3049e8",
                "InstanceType": "t3.micro",
                "HealthStatus": "Healthy",
                "LifecycleState": "InService"
            }
	],
	"CreatedTime": "2023-03-09T22:15:11.611Z",
	"SuspendedProcesses": [],
	"VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
	"EnabledMetrics": [],
	"Tags": [
            {
		"ResourceId": "group-1",
		"ResourceType": "auto-scaling-group",
		"Key": "environment",
		"Value": "production",
		"PropagateAtLaunch": true
            }
        ],
	"TerminationPolicies": [
	    "Default"
	],
	"NewInstancesProtectedFromScaleIn": false,
	"ServiceLinkedRoleARN": "arn"
    },

    ... additional groups ...

]
```

The following example expands on the previous example by removing everything except the names of the Auto Scaling groups that use launch configurations from the output:

```
aws autoscaling describe-auto-scaling-groups --region REGION \
  --query 'AutoScalingGroups[?LaunchConfigurationName!=`null`].AutoScalingGroupName'
```

The following shows example output\.

```
[
    "group-1",
    "group-2",
    "group-3"
]
```

Alternatively, to return the Auto Scaling group names with the names of their respective launch configurations and tags in the output, run the following command:

```
aws autoscaling describe-auto-scaling-groups --region REGION \
  --query 'AutoScalingGroups[?LaunchConfigurationName!=`null`].{AutoScalingGroupName: AutoScalingGroupName, LaunchConfigurationName: LaunchConfigurationName, Tags: Tags}'
```

The following shows example output\.

```
[
    {
        "AutoScalingGroupName": "group-1",
        "LaunchConfigurationName": "my-launch-config",
        "Tags": [
            {
                "ResourceId": "group-1",
                "ResourceType": "auto-scaling-group",
                "Key": "environment",
                "Value": "production",
                "PropagateAtLaunch": true
            }
        ]
    },

    ... additional groups ...

]
```