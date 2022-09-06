# Auto Scaling groups with multiple instance types and purchase options<a name="ec2-auto-scaling-mixed-instances-groups"></a>

You can launch and automatically scale a fleet of On\-Demand Instances and Spot Instances within a single Auto Scaling group\. In addition to receiving discounts for using Spot Instances, you can use Reserved Instances or a Savings Plan to receive discounted rates of the regular On\-Demand Instance pricing\. All of these factors combined help you to optimize your cost savings for EC2 instances, while making sure that you obtain the desired scale and performance for your application\.

You first specify the common configuration parameters in a launch template, and choose it when you create an Auto Scaling group\. When you configure the Auto Scaling group, you can:
+ Choose one or more instance types for the group \(optionally overriding the instance type that is specified by the launch template\)\.
+ Define multiple launch templates to allow instances with different CPU architectures \(for example, Arm and x86\) to launch in the same Auto Scaling group\.
+ Assign each instance type an individual weight\. Doing so might be useful, for example, if the instance types offer different vCPU, memory, storage, or network bandwidth capabilities\.
+ Prioritize instance types that can benefit from Savings Plan or Reserved Instance discount pricing\.
+ Specify how much On\-Demand and Spot capacity to launch, and specify an optional On\-Demand base portion\.
+ Define how Amazon EC2 Auto Scaling should distribute your On\-Demand and Spot capacity across instance types\.
+ Enable Capacity Rebalancing\. When you turn on Capacity Rebalancing, Amazon EC2 Auto Scaling attempts to launch a Spot Instance whenever the Amazon EC2 Spot service notifies that a Spot Instance is at an elevated risk of interruption\. After launching a new instance, it then terminates an old instance\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot Interruptions](ec2-auto-scaling-capacity-rebalancing.md)\.

You enhance availability by deploying your application across multiple instance types running in multiple Availability Zones\. You can use just one instance type, but it is a best practice to use a few instance types to allow Amazon EC2 Auto Scaling to launch another instance type in the event that there is insufficient instance capacity in your chosen Availability Zones\. With Spot Instances, if there is insufficient instance capacity, Amazon EC2 Auto Scaling keeps trying in other Spot Instance pools \(determined by your choice of instance types and allocation strategy\) rather than launching On\-Demand Instances, so that you can leverage the cost savings of Spot Instances\.

There are two ways to associate multiple instance types with your Auto Scaling group configuration:
+ Manually add instance types as described in this topic\.
+ Choose a set of instance attributes to use as criteria for selecting the instance types that your Auto Scaling group uses\. This is known as *attribute\-based instance type selection*\. For more information, see [Using attribute\-based instance type selection](create-asg-instance-type-requirements.md)\.

**Topics**
+ [Allocation strategies](#allocation-strategies)
+ [Control the proportion of On\-Demand Instances](#instances-distribution)
+ [Best practices for Spot Instances](#spot-best-practices)
+ [Prerequisites](#create-mixed-instances-group-prerequisites)
+ [Create an Auto Scaling group with Spot and On\-Demand Instances \(console\)](#create-mixed-instances-group-console)
+ [Configure Spot allocation strategies \(AWS CLI\)](#create-mixed-instances-group-aws-cli)
+ [Verify whether your Auto Scaling group is configured correctly and that the group has launched instances \(AWS CLI\)](#verify-launch-aws-cli)
+ [Configure overrides](ec2-auto-scaling-configuring-overrides.md)

## Allocation strategies<a name="allocation-strategies"></a>

The following allocation strategies determine how the Auto Scaling group fulfills your On\-Demand and Spot capacities from the possible instance types\. 

Amazon EC2 Auto Scaling first tries to ensure that your instances are evenly balanced across the Availability Zones that you specified\. Then, it launches instance types according to the allocation strategy that is specified\. 

### Spot Instances<a name="spot-allocation-strategy"></a>

Amazon EC2 Auto Scaling provides the following allocation strategies that can be used for Spot Instances: 

`capacity-optimized`  
Amazon EC2 Auto Scaling allocates your instances from the Spot Instance pool with optimal capacity for the number of instances that are launching\. Deploying in this way helps you make the most efficient use of spare EC2 capacity\.   
With Spot Instances, the pricing changes slowly over time based on long\-term trends in supply and demand, but capacity fluctuates in real time\. The `capacity-optimized` strategy automatically launches Spot Instances into the most available pools by looking at real\-time capacity data and predicting which are the most available\. This works well for workloads such as big data and analytics, image and media rendering, and machine learning\. It also works well for high performance computing that may have a higher cost of interruption associated with restarting work and checkpointing\. By offering the possibility of fewer interruptions, the `capacity-optimized` strategy can lower the overall cost of your workload\.  
Alternatively, you can use the `capacity-optimized-prioritized` allocation strategy and then set the order of instance types in the list of launch template overrides from highest to lowest priority \(first to last in the list\)\. Amazon EC2 Auto Scaling honors the instance type priorities on a best\-effort basis but optimizes for capacity first\. This is a good option for workloads where the possibility of disruption must be minimized, but also the preference for certain instance types matters\. 

`lowest-price`  
Amazon EC2 Auto Scaling allocates your Spot Instances from the N number of pools per Availability Zone that you specify and from the Spot Instance pools with the lowest price in each Availability Zone\.   
For example, if you specify four instance types and four Availability Zones, your Auto Scaling group has access to as many as 16 Spot pools \(four in each Availability Zone\)\. If you specify two Spot pools \(N=2\) for the allocation strategy, your Auto Scaling group can draw on the two cheapest pools per Availability Zone to fulfill your Spot capacity\.  
Note that Amazon EC2 Auto Scaling attempts to draw Spot Instances from the N number of pools that you specify on a best effort basis\. If a pool runs out of Spot capacity before fulfilling your desired capacity, Amazon EC2 Auto Scaling will continue to fulfill your request by drawing from the next cheapest pool\. To ensure that your desired capacity is met, you might receive Spot Instances from more than the N number of pools that you specified\. Similarly, if most of the pools have no Spot capacity, you might receive your full desired capacity from fewer than the N number of pools that you specified\.

To get started, we recommend choosing the `capacity-optimized` allocation strategy and specifying a few instance types that are appropriate for your application\. In addition, you can define a range of Availability Zones for Amazon EC2 Auto Scaling to choose from when launching instances\. 

Optionally, you can specify a maximum price for your Spot Instances\. If you don't specify a maximum price, the default maximum price is the On\-Demand price, but you still receive the steep discounts provided by Spot Instances\. These discounts are possible because of the stable Spot pricing that is made available using the new [Spot pricing model](https://aws.amazon.com/blogs/compute/new-amazon-ec2-spot-pricing/)\.

For more information about the allocation strategies for Spot Instances, see [Introducing the capacity\-optimized allocation strategy for Amazon EC2 Spot Instances](http://aws.amazon.com/blogs/compute/introducing-the-capacity-optimized-allocation-strategy-for-amazon-ec2-spot-instances/) in the AWS blog\.

### On\-Demand Instances<a name="on-demand-allocation-strategy"></a>

Amazon EC2 Auto Scaling provides the following allocation strategies that can be used for On\-Demand Instances: 

`lowest-price`  
Amazon EC2 Auto Scaling automatically deploys the cheapest instance type in each Availability Zone based on the current On\-Demand price\.  
To ensure that your desired capacity is met, you might receive On\-Demand Instances of more than one instance type in each Availability Zone, depending on how much capacity you request\.  
Currently, Amazon EC2 Auto Scaling does not honor the `lowest-price` allocation strategy for On\-Demand Instances when implementing your [termination policies](ec2-auto-scaling-termination-policies.md) during scale\-in events\.

`prioritized`  
Amazon EC2 Auto Scaling uses the order of instance types in the list of launch template overrides to determine which instance type to use first when fulfilling On\-Demand capacity\. For example, let's say that you specified three launch template overrides in the following order: `c5.large`, `c4.large`, and `c3.large`\. When your On\-Demand Instances are launched, the Auto Scaling group fulfills On\-Demand capacity by starting with `c5.large`, then `c4.large`, and then `c3.large`\.   
Consider the following when managing the priority order of your On\-Demand Instances:  
You can pay for usage upfront to get significant discounts for On\-Demand Instances by using either Savings Plans or Reserved Instances\. For more information about Savings Plans or Reserved Instances, see the [Amazon EC2 pricing](https://aws.amazon.com/ec2/pricing/) page\.   
+ With Reserved Instances, your discounted rate of the regular On\-Demand Instance pricing applies if Amazon EC2 Auto Scaling launches matching instance types\. That means that if you have unused Reserved Instances for `c4.large`, you can set the instance type priority to give the highest priority for your Reserved Instances to a `c4.large` instance type\. When a `c4.large` instance launches, you receive the Reserved Instance pricing\. 
+ With Savings Plans, your discounted rate of the regular On\-Demand Instance pricing applies when using either Amazon EC2 Instance Savings Plans or Compute Savings Plans\. Because of the flexible nature of Savings Plans, you have greater flexibility in prioritizing your instance types\. As long as you use instance types that are covered by your Savings Plan, you can set them in any order of priority and even occasionally change their order entirely, and continue to receive the discounted rate provided by your Savings Plan\. To learn more about Savings Plans, see the [Savings Plans User Guide](https://docs.aws.amazon.com/savingsplans/latest/userguide/)\.

## Control the proportion of On\-Demand Instances<a name="instances-distribution"></a>

You have full control over the proportion of instances in the Auto Scaling group that are launched as On\-Demand Instances\. To ensure that you always have instance capacity, you can designate a percentage of the group to launch as On\-Demand Instances and, optionally, a base number of On\-Demand Instances to start with\. If you choose to specify a base capacity of On\-Demand Instances, Amazon EC2 Auto Scaling launches Spot Instances only after launching this base capacity of On\-Demand Instances when the group scales out\. Anything beyond the base capacity uses the On\-Demand percentage to determine how many On\-Demand Instances and Spot Instances to launch\. You can specify any number from 0 to 100 for the On\-Demand percentage\. 

Amazon EC2 Auto Scaling converts the percentage to the equivalent number of instances\. If the result creates a fractional number, Amazon EC2 Auto Scaling rounds up to the next integer in favor of On\-Demand Instances\.

The behavior of the Auto Scaling group as it increases in size is as follows:


**Example: Scaling behavior**  

| Instances distribution | Total number of running instances across purchase options | 
| --- |--- |
|  | **10** | **20** | **30** | **40** | 
| --- |--- |--- |--- |--- |
| Example 1 |  |  |  |  | 
| On\-Demand base: 10 | 10 | 10 | 10 | 10 | 
| On\-Demand percentage above base: 50% | 0 | 5 | 10 | 15 | 
| Spot percentage: 50% | 0 | 5 | 10 | 15 | 
| Example 2 |  |  |  |  | 
| On\-Demand base: 0 | 0 | 0 | 0 | 0 | 
| On\-Demand percentage above base: 0% | 0 | 0 | 0 | 0 | 
| Spot percentage: 100% | 10 | 20 | 30 | 40 | 
| Example 3 |  |  |  |  | 
| On\-Demand base: 0 | 0 | 0 | 0 | 0 | 
| On\-Demand percentage above base: 60% | 6 | 12 | 18 | 24 | 
| Spot percentage: 40% | 4 | 8 | 12 | 16 | 
| Example 4 |  |  |  |  | 
| On\-Demand base: 0 | 0 | 0 | 0 | 0 | 
| On\-Demand percentage above base: 100% | 10 | 20 | 30 | 40 | 
| Spot percentage: 0% | 0 | 0 | 0 | 0 | 
| Example 5 |  |  |  |  | 
| On\-Demand base: 12 | 10 | 12 | 12 | 12 | 
| On\-Demand percentage above base: 0% | 0 | 0 | 0 | 0 | 
| Spot percentage: 100% | 0 | 8 | 18 | 28 | 

## Best practices for Spot Instances<a name="spot-best-practices"></a>

Before you create your Auto Scaling group to request Spot Instances, review [Best practices for EC2 Spot](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) in the *Amazon EC2 User Guide for Linux Instances*\. Use these best practices when you plan your request so that you can provision the type of instances that you want at the lowest possible price\. We also recommend that you do the following: 
+ Use the default maximum price, which is the On\-Demand price\. You pay only the Spot price for the Spot Instances that you launch\. If the Spot price is within your maximum price, whether your request is fulfilled depends on availability\. For more information, see [Pricing and savings](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html#spot-pricing) in the *Amazon EC2 User Guide for Linux Instances*\. 
+ Create your Auto Scaling group with multiple instance types\. Because capacity fluctuates independently for each instance type in an Availability Zone, you can often get more compute capacity when you have instance type flexibility\.
+ Similarly, don't limit yourself to only the most popular instance types\. Because prices adjust based on long\-term demand, popular instance types \(such as recently launched instance families\), tend to have more price adjustments\. Picking older\-generation instance types that are less popular tends to result in lower costs and fewer interruptions\.
+ We recommend that you use the `capacity-optimized` or `capacity-optimized-prioritized` allocation strategy\. This means that Amazon EC2 Auto Scaling launches instances using Spot pools that are optimally chosen based on the available Spot capacity, which helps you reduce the possibility of a Spot interruption\. 
+ If you choose the `lowest-price` allocation strategy and you run a web service, specify a high number of Spot pools, for example, N=10\. Specifying a high number of Spot pools reduces the impact of Spot Instance interruptions if a pool in one of the Availability Zones becomes temporarily unavailable\. If you run batch processing or other non\-mission critical applications, you can specify a lower number of Spot pools, for example, N=2\. This helps to ensure that you provision Spot Instances from only the very lowest priced Spot pools available per Availability Zone\. 

If you intend to specify a maximum price, use the AWS CLI or an SDK to create the Auto Scaling group, but be cautious\. If your maximum price is lower than the Spot price for the instance types that you selected, your Spot Instances are not launched\. 

## Prerequisites<a name="create-mixed-instances-group-prerequisites"></a>

Create a launch template\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\.

Verify that you have the permissions required to use a launch template\. Your `ec2:RunInstances` permissions are checked when use a launch template\. Your `iam:PassRole` permissions are also checked if the launch template specifies an IAM role\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

## Create an Auto Scaling group with Spot and On\-Demand Instances \(console\)<a name="create-mixed-instances-group-console"></a>

Complete the following steps to create a fleet of Spot Instances and On\-Demand Instances that you can scale\.

Before you begin, confirm that you have created a launch template, as described in [Prerequisites](#create-mixed-instances-group-prerequisites)\.

Verify that the launch template doesn't already request Spot Instances\. 

**To create an Auto Scaling group with Spot and On\-Demand Instances**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch template\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, do the following:

   1. For **Auto Scaling group name**, enter a name for your Auto Scaling group\.

   1. For **Launch template**, choose an existing launch template\.

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\. 

   1. Verify that your launch template supports all of the options that you are planning to use, and then choose **Next**\.

1. On the **Choose instance launch options** page, under **Network**, for **VPC**, choose a VPC\. The Auto Scaling group must be created in the same VPC as the security group you specified in your launch template\.

1. For **Availability Zones and subnets**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information, see [Considerations when choosing VPC subnets](asg-in-vpc.md#as-vpc-considerations)\.

1. For **Instance type requirements**, choose **Override launch template**, **Manually add instance types**\.

   1. For **Instance types**, choose which types of instances can be launched\. You can use our recommendations as a starting point\. 

   1. \(Optional\) To change the order of the instance types, use the arrows\. The order in which you set the instance types sets their launch priority if you choose a priority\-based allocation strategy\.

   1. \(Optional\) To use [instance weighting](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md), assign each instance type a relative weight that corresponds to how much the instance should count toward the capacity of the Auto Scaling group\.

1. Under **Instance purchase options**, make updates to the purchase options as needed to lower the cost of your application by using Spot Instances\.

   1. For **Instances distribution**, specify the percentages of On\-Demand Instances to Spot Instances to launch for the Auto Scaling group\. If your application is stateless, fault tolerant and can handle an instance being interrupted, you can specify a higher percentage of Spot Instances\.

   1. Depending on whether you chose to launch Spot Instances, you can select the check box next to **Include On\-Demand base capacity** and then specify the minimum amount of the Auto Scaling group's initial capacity that must be fulfilled by On\-Demand Instances\. Anything beyond the base capacity uses the **Instances distribution** settings to determine how many On\-Demand Instances and Spot Instances to launch\. 

1. Under **Allocation strategies**, for **On\-Demand allocation strategy**, choose an allocation strategy\.

1. For **Spot allocation strategy**, choose an allocation strategy\. We recommend that you keep the default setting of **Capacity optimized**\. If you prefer not to keep the default, choose **Lowest price**, and then enter the number of lowest priced Spot Instance pools to diversify across\. 

1. For **Capacity rebalance**, choose whether to enable or disable Capacity Rebalancing\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot Interruptions](ec2-auto-scaling-capacity-rebalancing.md)\. 

1. Choose **Next**\. 

   Or, you can accept the rest of the defaults, and choose **Skip to review**\. 

1. On the **Configure advanced options** page, configure the options as desired, and then choose **Next**:

1. \(Optional\) On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. For **Desired capacity**, enter the initial number of instances to launch\. When you change this number to a value outside of the minimum or maximum capacity limits, you must update the values of **Minimum capacity** or **Maximum capacity**\. For more information, see [Set capacity limits on your Auto Scaling group](asg-capacity-limits.md)\. 

   1. To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target tracking scaling policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)\.

   1. Under **Instance scale\-in protection**, choose whether to enable instance scale\-in protection\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

1. \(Optional\) To receive notifications, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Get Amazon SNS notifications when your Auto Scaling group scales](ec2-auto-scaling-sns-notifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tag Auto Scaling groups and instances](ec2-auto-scaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

## Configure Spot allocation strategies \(AWS CLI\)<a name="create-mixed-instances-group-aws-cli"></a>

The following example configurations show how to launch Spot Instances using the different Spot allocation strategies\.

**Note**  
These examples show how to use a configuration file formatted in JSON or YAML\. If you use AWS CLI version 1, you must specify a JSON\-formatted configuration file\. If you use AWS CLI version 2, you can specify a configuration file formatted in either YAML or JSON\.

**Topics**
+ [Example 1: Launch Spot Instances using the `capacity-optimized` allocation strategy](#capacity-optimized-aws-cli)
+ [Example 2: Launch Spot Instances using the `capacity-optimized-prioritized` allocation strategy](#capacity-optimized-prioritized-aws-cli)
+ [Example 3: Launch Spot Instances using the `lowest-price` allocation strategy diversified over two pools](#lowest-price-aws-cli)

### Example 1: Launch Spot Instances using the `capacity-optimized` allocation strategy<a name="capacity-optimized-aws-cli"></a>

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following:
+ The percentage of the group to launch as On\-Demand Instances \(`0`\) and a base number of On\-Demand Instances to start with \(`1`\)
+ The instance types to launch in priority order \(`c5.large`, `c5a.large`, `m5.large`, `m5a.large`, `c4.large`, `m4.large`, `c3.large`, `m3.large`\) 
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\), each corresponding to a different Availability Zone
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Default`\)

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c5.large` instance type first\. The Spot Instances come from the optimal Spot pool in each Availability Zone based on Spot Instance capacity\.

#### JSON<a name="capacity-optimized-aws-cli-json"></a>

```
aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
```

The following is an example `config.json` file\. 

```
{
    "AutoScalingGroupName": "my-asg",
    "MixedInstancesPolicy": {
        "LaunchTemplate": {
            "LaunchTemplateSpecification": {
                "LaunchTemplateName": "my-launch-template",
                "Version": "$Default"
            },
            "Overrides": [
                {
                    "InstanceType": "c5.large"
                },
                {
                    "InstanceType": "c5a.large"
                },
                {
                    "InstanceType": "m5.large"
                },
                {
                    "InstanceType": "m5a.large"
                },
                {
                    "InstanceType": "c4.large"
                },
                {
                    "InstanceType": "m4.large"
                },
                {
                    "InstanceType": "c3.large"
                },
                {
                    "InstanceType": "m3.large"
                }
            ]
        },
        "InstancesDistribution": {
            "OnDemandBaseCapacity": 1,
            "OnDemandPercentageAboveBaseCapacity": 0,
            "SpotAllocationStrategy": "capacity-optimized"
        }
    },
    "MinSize": 1,
    "MaxSize": 5,
    "DesiredCapacity": 3,
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782"
}
```

#### YAML<a name="capacity-optimized-aws-cli-yaml"></a>

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group, referencing a YAML file as the sole parameter for your Auto Scaling group instead of a JSON file\.

```
aws autoscaling create-auto-scaling-group --cli-input-yaml file://~/config.yaml
```

The following is an example `config.yaml` file\. 

```
---
AutoScalingGroupName: my-asg
MixedInstancesPolicy:
  LaunchTemplate:
    LaunchTemplateSpecification:
      LaunchTemplateName: my-launch-template
      Version: $Default
    Overrides:
    - InstanceType: c5.large
    - InstanceType: c5a.large
    - InstanceType: m5.large
    - InstanceType: m5a.large
    - InstanceType: c4.large
    - InstanceType: m4.large
    - InstanceType: c3.large
    - InstanceType: m3.large
  InstancesDistribution:
    OnDemandBaseCapacity: 1
    OnDemandPercentageAboveBaseCapacity: 0
    SpotAllocationStrategy: capacity-optimized
MinSize: 1
MaxSize: 5
DesiredCapacity: 3
VPCZoneIdentifier: subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782
```

### Example 2: Launch Spot Instances using the `capacity-optimized-prioritized` allocation strategy<a name="capacity-optimized-prioritized-aws-cli"></a>

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following:
+ The percentage of the group to launch as On\-Demand Instances \(`0`\) and a base number of On\-Demand Instances to start with \(`1`\)
+ The instance types to launch in priority order \(`c5.large`, `c5a.large`, `m5.large`, `m5a.large`, `c4.large`, `m4.large`, `c3.large`, `m3.large`\) 
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\), each corresponding to a different Availability Zone
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Latest`\)

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c5.large` instance type first\. When Amazon EC2 Auto Scaling attempts to fulfill your Spot capacity, it honors the instance type priorities on a best\-effort basis, but optimizes for capacity first\.

#### JSON<a name="capacity-optimized-prioritized-aws-cli-json"></a>

```
aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
```

The following is an example `config.json` file\. 

```
{
    "AutoScalingGroupName": "my-asg",
    "MixedInstancesPolicy": {
        "LaunchTemplate": {
            "LaunchTemplateSpecification": {
                "LaunchTemplateName": "my-launch-template",
                "Version": "$Latest"
            },
            "Overrides": [
                {
                    "InstanceType": "c5.large"
                },
                {
                    "InstanceType": "c5a.large"
                },
                {
                    "InstanceType": "m5.large"
                },
                {
                    "InstanceType": "m5a.large"
                },
                {
                    "InstanceType": "c4.large"
                },
                {
                    "InstanceType": "m4.large"
                },
                {
                    "InstanceType": "c3.large"
                },
                {
                    "InstanceType": "m3.large"
                }
            ]
        },
        "InstancesDistribution": {
            "OnDemandBaseCapacity": 1,
            "OnDemandPercentageAboveBaseCapacity": 0,
            "SpotAllocationStrategy": "capacity-optimized-prioritized"
        }
    },
    "MinSize": 1,
    "MaxSize": 5,
    "DesiredCapacity": 3,
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782"
}
```

#### YAML<a name="capacity-optimized-prioritized-aws-cli-yaml"></a>

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group, referencing a YAML file as the sole parameter for your Auto Scaling group instead of a JSON file\. 

```
aws autoscaling create-auto-scaling-group --cli-input-yaml file://~/config.yaml
```

The following is an example `config.yaml` file\. 

```
---
AutoScalingGroupName: my-asg
MixedInstancesPolicy:
  LaunchTemplate:
    LaunchTemplateSpecification:
      LaunchTemplateName: my-launch-template
      Version: $Default
    Overrides:
    - InstanceType: c5.large
    - InstanceType: c5a.large
    - InstanceType: m5.large
    - InstanceType: m5a.large
    - InstanceType: c4.large
    - InstanceType: m4.large
    - InstanceType: c3.large
    - InstanceType: m3.large
  InstancesDistribution:
    OnDemandBaseCapacity: 1
    OnDemandPercentageAboveBaseCapacity: 0
    SpotAllocationStrategy: capacity-optimized-prioritized
MinSize: 1
MaxSize: 5
DesiredCapacity: 3
VPCZoneIdentifier: subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782
```

### Example 3: Launch Spot Instances using the `lowest-price` allocation strategy diversified over two pools<a name="lowest-price-aws-cli"></a>

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following:
+ The percentage of the group to launch as On\-Demand Instances \(`50`\) without also specifying a base number of On\-Demand Instances to start with
+ The instance types to launch in priority order \(`c5.large`, `c5a.large`, `m5.large`, `m5a.large`, `c4.large`, `m4.large`, `c3.large`, `m3.large`\) 
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\), each corresponding to a different Availability Zone
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Latest`\)

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c5.large` instance type first\. For your Spot capacity, Amazon EC2 Auto Scaling attempts to launch the Spot Instances evenly across the two lowest\-priced pools in each Availability Zone\. 

#### JSON<a name="lowest-price-aws-cli-json"></a>

```
aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
```

The following is an example `config.json` file\. 

```
{
    "AutoScalingGroupName": "my-asg",
    "MixedInstancesPolicy": {
        "LaunchTemplate": {
            "LaunchTemplateSpecification": {
                "LaunchTemplateName": "my-launch-template",
                "Version": "$Latest"
            },
            "Overrides": [
                {
                    "InstanceType": "c5.large"
                },
                {
                    "InstanceType": "c5a.large"
                },
                {
                    "InstanceType": "m5.large"
                },
                {
                    "InstanceType": "m5a.large"
                },
                {
                    "InstanceType": "c4.large"
                },
                {
                    "InstanceType": "m4.large"
                },
                {
                    "InstanceType": "c3.large"
                },
                {
                    "InstanceType": "m3.large"
                }
            ]
        },
        "InstancesDistribution": {
            "OnDemandPercentageAboveBaseCapacity": 50,
            "SpotAllocationStrategy": "lowest-price",
            "SpotInstancePools": 2
        }
    },
    "MinSize": 1,
    "MaxSize": 5,
    "DesiredCapacity": 3,
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782"
}
```

#### YAML<a name="lowest-price-aws-cli-yaml"></a>

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group, referencing a YAML file as the sole parameter for your Auto Scaling group instead of a JSON file\. 

```
aws autoscaling create-auto-scaling-group --cli-input-yaml file://~/config.yaml
```

The following is an example `config.yaml` file\. 

```
---
AutoScalingGroupName: my-asg
MixedInstancesPolicy:
  LaunchTemplate:
    LaunchTemplateSpecification:
      LaunchTemplateName: my-launch-template
      Version: $Default
    Overrides:
    - InstanceType: c5.large
    - InstanceType: c5a.large
    - InstanceType: m5.large
    - InstanceType: m5a.large
    - InstanceType: c4.large
    - InstanceType: m4.large
    - InstanceType: c3.large
    - InstanceType: m3.large
  InstancesDistribution:
    OnDemandPercentageAboveBaseCapacity: 50
    SpotAllocationStrategy: lowest-price
    SpotInstancePools: 2
MinSize: 1
MaxSize: 5
DesiredCapacity: 3
VPCZoneIdentifier: subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782
```

**Note**  
For additional examples, see [Specify a different launch template for an instance type](ec2-auto-scaling-mixed-instances-groups-launch-template-overrides.md), [Configure instance weighting for Amazon EC2 Auto Scaling](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md), and [Use Capacity Rebalancing to handle Amazon EC2 Spot Interruptions](ec2-auto-scaling-capacity-rebalancing.md)\. 

## Verify whether your Auto Scaling group is configured correctly and that the group has launched instances \(AWS CLI\)<a name="verify-launch-aws-cli"></a>

To check whether your Auto Scaling group is configured correctly and that it has launched instances, use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\. Verify that the mixed instances policy and the list of subnets exist and are configured correctly\. If the instances have launched, you will see a list of the instances and their statuses\. To view the scaling activities that result from instances launching, use the [describe\-scaling\-activities](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command\. You can monitor scaling activities that are in progress and that are recently completed\.