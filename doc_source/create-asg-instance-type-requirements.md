# Create an Auto Scaling group using attribute\-based instance type selection<a name="create-asg-instance-type-requirements"></a>

When you create an Auto Scaling group, you must specify the necessary information to configure the Amazon EC2 instances, the Availability Zones and VPC subnets for the instances, the desired capacity, and the minimum and maximum capacity limits\.

As an alternative to manually choosing instance types when creating a [mixed instances group](ec2-auto-scaling-mixed-instances-groups.md), you can specify a set of instance attributes that describe your compute requirements\. As Amazon EC2 Auto Scaling launches instances, any instance types used by the Auto Scaling group must match your required instance attributes\. This is known as *attribute\-based instance type selection*\.

Your Auto Scaling group or your launch template specifies your instance attributes, including the amount of memory and computing power that you need for the applications that you plan to run on the instances\. Additionally, your Auto Scaling group or your launch template specifies two price protection thresholds for Spot and On\-Demand Instances, that you can optionally customize, so that you can prevent Amazon EC2 Auto Scaling from launching more expensive instance types if you don’t need them\.

This approach is ideal for workloads and frameworks that can be flexible about which instance types they use, such as containers, big data, and CI/CD\.

The following are benefits of attribute\-based instance type selection:
+ Amazon EC2 Auto Scaling can select from a wide range of instance types for launching Spot Instances\. This meets the Spot best practice of being flexible about instance types, which gives the Amazon EC2 Spot service a better chance of finding and allocating your required amount of compute capacity\.
+ With so many options available, finding the right instance types for your workload can be time consuming\. By specifying instance attributes, you can simplify instance type selection when configuring a mixed instances group\. 
+ Your Auto Scaling groups can use newer generation instance types as they're released\. Newer generation instance types are automatically used when they match your requirements and align with the allocation strategies you choose for your Auto Scaling group\. 

You can use attribute\-based instance type selection through the AWS Management Console, AWS CLI, or SDKs\.

For information about how to configure attribute\-based instance type selection in a launch template, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\. For information about how to configure attribute\-based instance type selection by passing parameters in Amazon EC2 Auto Scaling API calls using an SDK, see [InstanceRequirements](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_InstanceRequirements.html) in the *Amazon EC2 Auto Scaling API Reference*\.

To learn more about attribute\-based instance type selection, see [Attribute\-Based Instance Type Selection for EC2 Auto Scaling and EC2 Fleet](http://aws.amazon.com/blogs/aws/new-attribute-based-instance-type-selection-for-ec2-auto-scaling-and-ec2-fleet/) on the AWS Blog\.

**Topics**
+ [Considerations](#use-attribute-based-instance-type-selection-considerations)
+ [Prerequisites](#use-attribute-based-instance-type-selection-prerequisites)
+ [Use attribute\-based instance type selection](#use-attribute-based-instance-type-selection)
+ [Limitations](#use-attribute-based-instance-type-selection-limitations)

## Considerations<a name="use-attribute-based-instance-type-selection-considerations"></a>

The following are things to consider when using attribute\-based instance type selection:
+ For most general purpose workloads, it is enough to specify the number of vCPUs and memory that you need\. For advanced use cases, you can specify attributes like storage type, network interfaces, CPU manufacturer, and accelerator type\.
+ By default, you set the value for the desired capacity of your Auto Scaling group as the number of instances\. You can optionally specify the desired capacity type as the number of vCPUs or the amount of memory when you use attribute\-based instance type selection\. Then, when Amazon EC2 Auto Scaling launches instances, their number of vCPUs or amount of memory count toward your desired capacity\. When creating your group in the Amazon EC2 Auto Scaling console, this setting appears in the **Group size** section on the **Configure group size and scaling policies** page\. This feature is a useful replacement for the [instance weighting](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md) feature\.
+ You can preview the instance types that match your compute requirements without launching them and adjust your requirements if necessary\. When creating your Auto Scaling group in the Amazon EC2 Auto Scaling console, a preview of the instance types appears in the **Preview matching instance types** section on the **Choose instance launch options** page\.
+ Alternatively, you can preview the instance types by making an Amazon EC2 [GetInstanceTypesFromInstanceRequirements](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_GetInstanceTypesFromInstanceRequirements.html) API call using the AWS CLI or an SDK\. Pass the `InstanceRequirements` parameters in the request in the exact format that you would use to create or update an Auto Scaling group\. For more information, see [Preview instance types with specified attributes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet-attribute-based-instance-type-selection.html#spotfleet-get-instance-types-from-instance-requirements) in the *Amazon EC2 User Guide for Linux Instances*\.

### Understand price protection<a name="understand-price-protection"></a>

Price protection is a feature that protects your Auto Scaling group from extreme price differences across instance types\. When you create a new Auto Scaling group or update an existing Auto Scaling group with attribute\-based instance type selection, we enable price protection by default\. You can optionally choose your price protection thresholds for Spot and On\-Demand Instances, and Amazon EC2 Auto Scaling doesn't select instance types whose price is higher than your specified thresholds\. The thresholds represent what you are willing to pay, defined in terms of a percentage above a baseline, rather than as absolute values\. The baseline is determined by the price of the least expensive M, C, or R instance type with your specified attributes\. If your attributes don't match any M, C, or R instance types, we use the lowest priced instance type\.

If you don't specify a threshold, the following are used by default:
+ For On\-Demand Instances, the price protection threshold is set to 20 percent\.
+ For Spot Instances, the price protection threshold is set to 100 percent\.

To update these values when creating your Auto Scaling group in the Amazon EC2 Auto Scaling console, on the **Choose instance launch options** page, choose the desired price protection attribute from the **Additional instance attributes** drop\-down list, and then type or choose a value for the attribute in the text box\. You can also update these values at any time by editing the Auto Scaling group from the console or passing the relevant parameters using the AWS CLI or an SDK\.

**Note**  
If you set **Desired capacity type** to **vCPUs** or **Memory GiB**, the price protection threshold is applied based on the per vCPU or per memory price instead of the per instance price\. 

## Prerequisites<a name="use-attribute-based-instance-type-selection-prerequisites"></a>

Create a launch template that includes the parameters required to launch an EC2 instance, such as the Amazon Machine Image \(AMI\) and security groups\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\.

Verify that you have the permissions required to use a launch template\. Your `ec2:RunInstances` permissions are checked when use a launch template\. Your `iam:PassRole` permissions are also checked if the launch template specifies an IAM role\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

## Use attribute\-based instance type selection<a name="use-attribute-based-instance-type-selection"></a>

Complete the following steps to create an Auto Scaling group that uses attribute\-based instance type selection\.

Before you begin, confirm that you have created a launch template, as described in [Prerequisites](#use-attribute-based-instance-type-selection-prerequisites)\.

Verify that the launch template doesn't already request Spot Instances\. 

**To create an Auto Scaling group using attribute\-based instance type selection \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch template\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, do the following:

   1. For **Auto Scaling group name**, enter a name for your Auto Scaling group\.

   1. For **Launch template**, choose an existing launch template\.

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\. 

   1. Verify that your launch template supports all of the options that you are planning to use, and then choose **Next**\.

1. On the **Choose instance launch options** page, under **Network**, for **VPC**, choose a VPC\. The Auto Scaling group must be created in the same VPC as the security group you specified in your launch template\.

1. For **Availability Zones and subnets**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information, see [Considerations when choosing VPC subnets](asg-in-vpc.md#as-vpc-considerations)\.

1. For **Instance type requirements**, choose **Override launch template**\.
**Note**  
If you are already using attribute\-based instance type selection in your launch template, the settings that you defined in your launch template, such as vCPUs and memory, are automatically populated\.

   1. Under **Specify instance attributes**, start by entering your vCPUs and memory requirements\.
      + **vCPUs**: Enter the minimum and maximum number of vCPUs for your compute requirements\. Select the **No minimum** or **No maximum** check box to indicate no limit in that direction\.
      + **Memory \(MiB\)**: Enter the minimum and maximum amount of memory, in MiB, for your compute requirements\. Select the **No minimum** or **No maximum** check box to indicate no limit in that direction\.

   1. \(Optional\) For **Additional instance attributes**, choose **Add attribute** to express your compute requirements in more detail\. The attributes and values that you choose here further limit which instance types may be launched\. For a complete list of supported attributes, see [InstanceRequirements](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_InstanceRequirements.html) in the *Amazon EC2 Auto Scaling API Reference*\.

   1. For **Preview matching instance types**, view the instance types that match the specified compute requirements, such as vCPUs, memory, and storage\.

      \(Optional\) To exclude instance types, select the instance types to exclude and choose **Exclude selected instance types**\. Alternatively, for **Additional instance attributes**, choose **Add attribute**, **Exclude instance types** and then select the instance types to exclude\.

1. Under **Instance purchase options**, make updates to the purchase options as needed to lower the cost of your application by using Spot Instances\.

   1. For **Instances distribution**, specify the percentages of On\-Demand Instances to Spot Instances to launch for the Auto Scaling group\. If your application is stateless, fault tolerant and can handle an instance being interrupted, you can specify a higher percentage of Spot Instances\.

   1. Depending on whether you chose to launch Spot Instances, you can select the check box next to **Include On\-Demand base capacity** and then specify the minimum amount of the Auto Scaling group's initial capacity that must be fulfilled by On\-Demand Instances\. Anything beyond the base capacity uses the **Instances distribution** settings to determine how many On\-Demand Instances and Spot Instances to launch\. 

1. Under **Allocation strategies**, **Lowest price** is automatically selected for the **On\-Demand allocation strategy** and cannot be changed\.

1. For **Spot allocation strategy**, choose an allocation strategy\. We recommend that you keep the default setting of **Capacity optimized**\. If you prefer not to keep the default, choose **Lowest price**, and then enter the number of lowest priced Spot Instance pools to diversify across\. 

1. For **Capacity rebalance**, choose whether to enable or disable Capacity Rebalancing\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot Interruptions](ec2-auto-scaling-capacity-rebalancing.md)\. 

1. Choose **Next** twice to go to the **Configure group size and scaling policies** page\. 

   Or, you can accept the rest of the defaults, and choose **Skip to review**\. 

1. \(Optional\) On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. Choose a **Desired capacity type**\. **Units** \(default\), **vCPUs** and **Memory GiB** are supported\.

   1. Enter the initial size of your Auto Scaling group for **Desired capacity** and update the **Minimum capacity** and **Maximum capacity** limits as needed\. For more information, see [Set capacity limits on your Auto Scaling group](asg-capacity-limits.md)\.

   1. To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target Tracking Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

   1. Under **Instance scale\-in protection**, choose whether to enable instance scale\-in protection\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

1. \(Optional\) To receive notifications, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Get Amazon SNS notifications when your Auto Scaling group scales](ec2-auto-scaling-sns-notifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tag Auto Scaling groups and instances](ec2-auto-scaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

### Example: Create an Auto Scaling group using attribute\-based instance type selection \(AWS CLI\)<a name="attribute-based-instance-type-selection-aws-cli"></a>

To create an Auto Scaling group with attribute\-based instance type selection using the command line, you can use the following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command\. 

The following instance attributes are specified:
+ `VCpuCount` – The instance types must have a minimum of four vCPUs and a maximum of eight vCPUs\. 
+ `MemoryMiB` – The instance types must have a minimum of 16384 MiB of memory\. 
+ `CpuManufacturers` – The instance types must have an Intel manufactured CPU\. 

#### JSON<a name="attribute-based-instance-type-selection-aws-cli-json"></a>

```
aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
```

The following is an example `config.json` file\. 

```
{
    "AutoScalingGroupName": "my-asg",
    "DesiredCapacityType": "units",
    "MixedInstancesPolicy": {
        "LaunchTemplate": {
            "LaunchTemplateSpecification": {
                "LaunchTemplateName": "my-launch-template",
                "Version": "$Default"
            },
            "Overrides": [{
                "InstanceRequirements": {
                    "VCpuCount": {"Min": 4, "Max": 8},
                    "MemoryMiB": {"Min": 16384},
                    "CpuManufacturers": ["intel"]
                }
            }]
        },
        "InstancesDistribution": {
            "OnDemandPercentageAboveBaseCapacity": 50,
            "SpotAllocationStrategy": "capacity-optimized"
        }
    },
    "MinSize": 0,
    "MaxSize": 100,
    "DesiredCapacity": 4,
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782"
}
```

To set the value for desired capacity as the number of vCPUs or the amount of memory, specify `"DesiredCapacityType": "vcpu"` or `"DesiredCapacityType": "memory-mib"` in the file\. The default desired capacity type is `units`, which sets the value for desired capacity as the number of instances\.

#### YAML<a name="attribute-based-instance-type-selection-aws-cli-yaml"></a>

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group, referencing a YAML file as the sole parameter for your Auto Scaling group instead of a JSON file\.

```
aws autoscaling create-auto-scaling-group --cli-input-yaml file://~/config.yaml
```

The following is an example `config.yaml` file\. 

```
---
AutoScalingGroupName: my-asg
DesiredCapacityType: units
MixedInstancesPolicy:
  LaunchTemplate:
    LaunchTemplateSpecification:
      LaunchTemplateName: my-launch-template
      Version: $Default
    Overrides:
    - InstanceRequirements:
         VCpuCount:
           Min: 2
           Max: 4
         MemoryMiB:
           Min: 2048
         CpuManufacturers:
         - intel
  InstancesDistribution:
    OnDemandPercentageAboveBaseCapacity: 50
    SpotAllocationStrategy: capacity-optimized
MinSize: 0
MaxSize: 100
DesiredCapacity: 4
VPCZoneIdentifier: subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782
```

To set the value for desired capacity as the number of vCPUs or the amount of memory, specify `DesiredCapacityType: vcpu` or `DesiredCapacityType: memory-mib` in the file\. The default desired capacity type is `units`, which sets the value for desired capacity as the number of instances\.

## Limitations<a name="use-attribute-based-instance-type-selection-limitations"></a>
+ You can configure attribute\-based instance type selection only for Auto Scaling groups that use a launch template\.
+ If you have an existing Auto Scaling group and plan to substitute your instance types with your required instance attributes, your On\-Demand allocation strategy must be `lowest-price`\. To use the `prioritized` allocation strategy, you must continue to manually add and prioritize your instance types\. Additionally, your Spot allocation strategy must be either `capacity-optimized` or `lowest-price`\. To use the `capacity-optimized-prioritized` allocation strategy, you must manually add and prioritize your instance types\. 