# Auto Scaling groups with multiple instance types and purchase options<a name="ec2-auto-scaling-mixed-instances-groups"></a>

You can launch and automatically scale a fleet of On\-Demand Instances and Spot Instances within a single Auto Scaling group\. In addition to receiving discounts for using Spot Instances, you can use Reserved Instances or a Savings Plan to receive discounted rates of the regular On\-Demand Instance pricing\. All of these factors combined help you to optimize your cost savings for EC2 instances and help you get the desired scale and performance for your application\.

The following steps describe how to create the Auto Scaling group: 
+ Specify the launch template for launching the instances\.
+ Choose the VPC and subnets to launch your Auto Scaling group in\.
+ Choose the option to override the existing instance type requirements of the launch template with new requirements\.
+ Manually choose and prioritize your instance types\. For example, you might choose to prioritize instance types that can benefit from Savings Plan or Reserved Instance discount pricing for On\-Demand Instances\.
+ Specify the percentages of On\-Demand Instances and Spot Instances to launch\.
+ Choose allocation strategies that determine how Amazon EC2 Auto Scaling fulfills your On\-Demand and Spot capacities from the possible instance types\.
+ Specify your group size, including your desired capacity, minimum capacity, and maximum capacity\.

You enhance availability by deploying your application across multiple instance types running in multiple Availability Zones\. Although you can use one instance type, it's a best practice to use multiple instance types\. This way, Amazon EC2 Auto Scaling can launch another instance type if there is insufficient instance capacity in your chosen Availability Zones\. If there is insufficient instance capacity with Spot Instances, Amazon EC2 Auto Scaling keeps trying to launch from other Spot Instance pools\. \(The pools it uses are determined by your choice of instance types and allocation strategy\.\) Amazon EC2 Auto Scaling helps you leverage the cost savings of Spot Instances by launching them instead of On\-Demand Instances\.

**Topics**
+ [Allocation strategies](#allocation-strategies)
+ [Best practices for Spot Instances](#spot-best-practices)
+ [Control the proportion of On\-Demand Instances](#instances-distribution)
+ [Prerequisites](#create-mixed-instances-group-prerequisites)
+ [Create an Auto Scaling group with Spot and On\-Demand Instances \(console\)](#create-mixed-instances-group-console)
+ [Create an Auto Scaling group with Spot and On\-Demand Instances \(AWS CLI\)](#create-mixed-instances-group-aws-cli)
+ [Verify your Auto Scaling group configuration and launched instances \(AWS CLI\)](#verify-launch-aws-cli)
+ [Configure overrides](ec2-auto-scaling-configuring-overrides.md)

## Allocation strategies<a name="allocation-strategies"></a>

The following allocation strategies determine how the Auto Scaling group fulfills your On\-Demand and Spot capacities from the possible instance types\. 

First, Amazon EC2 Auto Scaling tries to balance your instances evenly across your specified Availability Zones\. Then, it launches instance types according to the specified allocation strategy\. 

### Spot Instances<a name="spot-allocation-strategy"></a>

Amazon EC2 Auto Scaling provides the following allocation strategies that can be used for Spot Instances: 

`capacity-optimized`  
Amazon EC2 Auto Scaling requests your Spot Instance from the pool with optimal capacity for the number of instances that are launching\.   
With Spot Instances, the pricing changes slowly over time based on long\-term trends in supply and demand, but capacity fluctuates in real time\. The `capacity-optimized` strategy automatically launches Spot Instances into the most available pools by looking at real\-time capacity data and predicting which are the most available\. This helps to minimize possible disruptions for workloads that might have a higher cost of interruption associated with restarting work and checkpointing\. To give certain instance types a higher chance of launching first, use `capacity-optimized-prioritized`\. 

`capacity-optimized-prioritized`  
You set the order of instance types for the launch template overrides from highest to lowest priority \(from first to last in the list\)\. Amazon EC2 Auto Scaling honors the instance type priorities on a best\-effort basis but optimizes for capacity first\. This is a good option for workloads where the possibility of disruption must be minimized, but also the preference for certain instance types matters\. If the On\-Demand allocation strategy is set to `prioritized`, the same priority is applied when fulfilling On\-Demand capacity\. 

`lowest-price`  
Amazon EC2 Auto Scaling requests your Spot Instances using the lowest priced pools within an Availability Zone, across the N number of pools of Spot pools that you specify for the **Lowest priced pools** setting\. For example, if you specify four instance types and four Availability Zones, your Auto Scaling group can access up to 16 Spot pools\. \(Four in each Availability Zone\.\) If you specify two Spot pools \(N=2\) for the allocation strategy, your Auto Scaling group can draw on the two lowest priced pools per Availability Zone to fulfill your Spot capacity\.  
Because this strategy only considers instance price and not capacity availability, it might lead to high interruption rates\.

`price-capacity-optimized` \(recommended\)  
The price and capacity optimized allocation strategy looks at both price and capacity to select the Spot Instance pools that are the least likely to be interrupted and have the lowest possible price\.

To get started, we recommend choosing the `price-capacity-optimized` allocation strategy and specifying a set of instance types that are appropriate for your application\. In addition, you can define a range of Availability Zones for Amazon EC2 Auto Scaling to choose from when launching instances\.

For more information about the `price-capacity-optimized` strategy and use cases where it's helpful, see [Introducing the price\-capacity\-optimized allocation strategy for EC2 Spot Instances](http://aws.amazon.com/blogs/compute/introducing-price-capacity-optimized-allocation-strategy-for-ec2-spot-instances/) in the AWS blog\.

Optionally, you can specify a maximum price for your Spot Instances\. If you don't specify a maximum price, the default maximum price is the On\-Demand price\. However, you still receive the steep discounts provided by Spot Instances\. These discounts are possible because of the stable Spot pricing that's available with the [Spot pricing model](https://aws.amazon.com/blogs/compute/new-amazon-ec2-spot-pricing/)\.

### On\-Demand Instances<a name="on-demand-allocation-strategy"></a>

Amazon EC2 Auto Scaling provides the following allocation strategies that can be used for On\-Demand Instances: 

`lowest-price`  
Amazon EC2 Auto Scaling automatically deploys the lowest priced instance type in each Availability Zone based on the current On\-Demand price\.  
To meet your desired capacity, you might receive On\-Demand Instances of more than one instance type in each Availability Zone\. This depends on how much capacity you request\.

`prioritized`  
When fulfilling On\-Demand capacity, Amazon EC2 Auto Scaling determines which instance type to use first based on the order of instance types in the list of launch template overrides\. For example, let's say that you specify three launch template overrides in the following order: `c5.large`, `c4.large`, and `c3.large`\. When your On\-Demand Instances launch, the Auto Scaling group fulfills On\-Demand capacity by starting with `c5.large`, followed by `c4.large`, and lastly `c3.large`\.   
Consider the following when managing the priority order of your On\-Demand Instances:  
You can pay for usage upfront to get significant discounts for On\-Demand Instances by using Savings Plans or Reserved Instances\. For more information, see the [Amazon EC2 pricing](https://aws.amazon.com/ec2/pricing/) page\.   
+ With Reserved Instances, your discounted rate of the regular On\-Demand Instance pricing applies if Amazon EC2 Auto Scaling launches matching instance types\. Therefore, if you have unused Reserved Instances for `c4.large`, you can set the instance type priority to give the highest priority for your Reserved Instances to a `c4.large` instance type\. When a `c4.large` instance launches, you receive the Reserved Instance pricing\. 
+ With Savings Plans, your discounted rate of the regular On\-Demand Instance pricing applies when using Amazon EC2 Instance Savings Plans or Compute Savings Plans\. With Savings Plans, you have more flexibility when prioritizing your instance types\. As long as you use instance types that are covered by your Savings Plan, you can set them in any priority order\. You can also occasionally change the entire order of your instance types, while still receiving the Savings Plan discounted rate\. For more information about Savings Plans, see the [Savings Plans User Guide](https://docs.aws.amazon.com/savingsplans/latest/userguide/)\.

## Best practices for Spot Instances<a name="spot-best-practices"></a>

Before you create your Auto Scaling group to request Spot Instances, review [Best practices for EC2 Spot](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) in the *Amazon EC2 User Guide for Linux Instances*\. Use these best practices to plan your request so that you can provision the type of instances that you want at the lowest possible price\. We also recommend that you do the following: 
+ Do not specify a maximum price\. Your application might not run if you do not receive any Spot Instances, such as when your maximum price is too low\. The default maximum price is the On\-Demand price, and you pay only the Spot price for the Spot Instances that you launch\. When the Spot price is lower than the maximum price, whether your request is fulfilled depends on availability\. For more information, see [Pricing and savings](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html#spot-pricing) in the *Amazon EC2 User Guide for Linux Instances*\. 
+ A good rule of thumb is to be flexible across at least 10 instance types for each workload\. In addition, make sure that all Availability Zones are configured for use in your VPC and selected for your workload\. Because capacity fluctuates independently for each instance type in an Availability Zone, you can often get more compute capacity when you have instance type flexibility\.
+ Don't limit yourself to the most popular new instance types\. Choosing older generation instance types tends to result in fewer interruptions because they are less in demand from On\-Demand customers\.
+ Enable Capacity Rebalancing\. When you do this, Amazon EC2 Auto Scaling attempts to launch a Spot Instance whenever the Amazon EC2 Spot service notifies that a Spot Instance is at an elevated risk of interruption\. After launching a new instance, it then terminates an old instance\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot interruptions](ec2-auto-scaling-capacity-rebalancing.md)\.
+ We recommend that you use the `price-capacity-optimized` allocation strategy instead of the `lowest-price` allocation strategy to avoid requesting Spot Instances from pools with a high chance of interruption\.
+ If you choose the `lowest-price` allocation strategy and you run a web service, specify a high number of Spot pools, such as N=10\. Doing this reduces the impact of Spot Instance interruptions if a pool in one of the Availability Zones becomes temporarily unavailable\. If you run batch processing or other non\-mission critical applications, you can specify a lower number of Spot pools, such as N=2\. This provisions Spot Instances from only the lowest priced Spot pools available per Availability Zone\. 

  Amazon EC2 Auto Scaling attempts to draw Spot Instances from the N number of pools that you specify on a best\-effort basis\. If a pool runs out of Spot capacity before fulfilling your desired capacity, Amazon EC2 Auto Scaling will continue to fulfill your request by drawing from the next lowest priced pool\. To meet your desired capacity, you might receive Spot Instances from more pools than your specified N number\. Likewise, if most of the pools have no Spot capacity, you might receive your full desired capacity from fewer pools than your specified N number\.

If you intend to specify a maximum price, use the AWS CLI or an SDK to create the Auto Scaling group, but be cautious\. If your maximum price is lower than the Spot price for the instance types that you selected, your Spot Instances are not launched\. 

## Control the proportion of On\-Demand Instances<a name="instances-distribution"></a>

You have full control over the proportion of instances in the Auto Scaling group that are launched as On\-Demand Instances\. To make sure that you always have instance capacity, you can designate a percentage of the group to launch as On\-Demand Instances\. Optionally, you can also designate a base number of On\-Demand Instances to start with\. If you designate a base capacity of On\-Demand Instances, Amazon EC2 Auto Scaling waits to launch Spot Instances until after it launches the base capacity of On\-Demand Instances when the group scales out\. Anything beyond the base capacity uses the On\-Demand percentage to determine how many On\-Demand Instances and Spot Instances to launch\. You can specify any number from 0 to 100 for the On\-Demand percentage\. 

Amazon EC2 Auto Scaling converts the percentage to the equivalent number of instances\. If the result creates a fractional number, Amazon EC2 Auto Scaling rounds up to the next integer in favor of On\-Demand Instances\.

The following table demonstrates the behavior of the Auto Scaling group as it increases in size\.


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

## Prerequisites<a name="create-mixed-instances-group-prerequisites"></a>

Create a launch template that includes the parameters required to launch an EC2 instance, such as the Amazon Machine Image \(AMI\) and security groups\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\.

Verify that the launch template doesn't already request Spot Instances\. 

Verify that you have the permissions required to use a launch template\. Your `ec2:RunInstances` permissions are checked when you use a launch template\. Your `iam:PassRole` permissions are also checked if the launch template specifies an IAM role\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

## Create an Auto Scaling group with Spot and On\-Demand Instances \(console\)<a name="create-mixed-instances-group-console"></a>

Complete the following steps to create a fleet of Spot Instances and On\-Demand Instances that you can scale\.

Use the following procedure to choose which individual instance types your group can launch\. If you prefer to use instance attributes as criteria for selecting instance types, configure instance type requirements by specifying the number of vCPUs and memory that you need\. For more information, see [Create an Auto Scaling group using attribute\-based instance type selection](create-asg-instance-type-requirements.md)\. You can also specify other attributes like storage type, network interfaces, CPU manufacturer, and accelerator type\.

**To create an Auto Scaling group with Spot and On\-Demand Instances**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch template\.

1. Choose **Create an Auto Scaling group**\. 

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group\.

1. To choose your launch template, do the following:

   1. For **Launch template**, choose an existing launch template\.

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\. 

   1. Verify that your launch template supports all of the options that you plan to use, and then choose **Next**\.

1. To choose a VPC and the subnets you want to launch your instances in, do the following:

   1. On the **Choose instance launch options** page, under **Network**, for **VPC**, choose a VPC\. The Auto Scaling group must be created in the same VPC as the security group you specified in your launch template\.

   1. For **Availability Zones and subnets**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information, see [Considerations when choosing VPC subnets](asg-in-vpc.md#as-vpc-considerations)\.

1. To manually choose which individual instance types your group can launch, do the following:

   1. For **Instance type requirements**, choose **Override launch template**, and then choose **Manually add instance types**\. 

   1. Choose your instance types\. You can use our recommendations as a starting point\. **Family and generation flexible** is selected by default\.

      To change the order of the instance types, use the arrows\. If you choose an allocation strategy that supports prioritization, the instance type order sets their launch priority\.

      To remove an instance type, choose **X**\.

      Optionally, for the boxes in the **Weight** column, assign each instance type a relative weight\. To do so, enter the number of units that an instance of that type counts toward the desired capacity of the group\. Doing so might be useful if the instance types offer different vCPU, memory, storage, or network bandwidth capabilities\. For more information, see [Configure instance weighting for Amazon EC2 Auto Scaling](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md)\. 
**Note**  
If you chose to use **Size flexible** recommendations, then all instance types that are part of this section automatically have a weight value\. If you don't want to specify any weights, clear the boxes in the **Weight** column for all instance types\.

   1. Under **Instance purchase options**, for **Instances distribution**, specify the percentages of the group to be launched as On\-Demand Instances and Spot Instances respectively\. If your application is stateless, fault tolerant and can handle an instance being interrupted, you can specify a higher percentage of Spot Instances\.

   1. When you specify a percentage for Spot Instances, you can select the check box next to **Include On\-Demand base capacity** and then specify the minimum amount of the Auto Scaling group's initial capacity that must be fulfilled by On\-Demand Instances\. Anything beyond the base capacity uses the **Instances distribution** settings to determine how many On\-Demand Instances and Spot Instances to launch\. 

   1. Under **Allocation strategies**, for **On\-Demand allocation strategy**, choose an allocation strategy\. When you manually choose your instance types, **Prioritized** is selected by default\.

   1. For **Spot allocation strategy**, choose an allocation strategy\. **Price capacity optimized** is selected by default\. **Lowest price** is hidden by default and only appears when you choose **Show all strategies**\.
**Note**  
If you chose **Lowest price**, enter the number of lowest priced pools to diversify across for **Lowest priced pools**\. If you chose **Capacity optimized**, you can optionally check the **Prioritize instance types** box to let Amazon EC2 Auto Scaling choose which instance type to launch first based on the order your instance types are listed in\. 

   1. For **Capacity rebalance**, choose whether to enable or disable Capacity Rebalancing\. 

      If you chose a percentage for Spot Instances, you can use Capacity Rebalancing to automatically respond when your Spot Instances approach termination from a Spot interruption\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot interruptions](ec2-auto-scaling-capacity-rebalancing.md)\. 

   1. When finished, choose **Next** twice to go to the **Configure group size and scaling policies** page\.

1. For the **Configure group size and scaling policies** step, do the following:

   1. Enter the initial size of your Auto Scaling group for **Desired capacity** and update the **Minimum capacity** and **Maximum capacity** limits as needed\. For more information, see [Set capacity limits on your Auto Scaling group](asg-capacity-limits.md)\.
**Note**  
By default, the desired capacity and minimum and maximum capacity limits are expressed as the number of instances\. If you assigned weights to your instance types, you must convert these values to the same unit of measurement that you used to assign weights, such as the number of vCPUs\. 

   1. \(Optional\) Configure the group to scale by specifying a target tracking scaling policy\. Alternatively, specify this policy after you create the group\. For more information, see [Target tracking scaling policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)\.

   1. \(Optional\) Enable instance scale\-in protection, which prevents your Auto Scaling group from terminating instances when scaling in\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

   1. When finished, choose **Next**\.

1. \(Optional\) To receive notifications when your group scales, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Get Amazon SNS notifications when your Auto Scaling group scales](ec2-auto-scaling-sns-notifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tag Auto Scaling groups and instances](ec2-auto-scaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

## Create an Auto Scaling group with Spot and On\-Demand Instances \(AWS CLI\)<a name="create-mixed-instances-group-aws-cli"></a>

The following example configurations show how to launch Spot Instances using the different Spot allocation strategies\.

**Note**  
These examples show how to use a configuration file formatted in JSON or YAML\. If you use AWS CLI version 1, you must specify a JSON\-formatted configuration file\. If you use AWS CLI version 2, you can specify a configuration file formatted in either YAML or JSON\.

**Topics**
+ [Example 1: Launch Spot Instances using the `capacity-optimized` allocation strategy](#capacity-optimized-aws-cli)
+ [Example 2: Launch Spot Instances using the `capacity-optimized-prioritized` allocation strategy](#capacity-optimized-prioritized-aws-cli)
+ [Example 3: Launch Spot Instances using the `lowest-price` allocation strategy diversified over two pools](#lowest-price-aws-cli)
+ [Example 4: Launch Spot Instances using the `price-capacity-optimized` allocation strategy](#price-capacity-optimized-aws-cli)

### Example 1: Launch Spot Instances using the `capacity-optimized` allocation strategy<a name="capacity-optimized-aws-cli"></a>

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following:
+ The percentage of the group to launch as On\-Demand Instances \(`0`\) and a base number of On\-Demand Instances to start with \(`1`\)\.
+ The instance types to launch in priority order \(`c5.large`, `c5a.large`, `m5.large`, `m5a.large`, `c4.large`, `m4.large`, `c3.large`, `m3.large`\) \.
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\)\. Each corresponds to a different Availability Zone\.
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Default`\)\.

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

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group\. This references a YAML file as the sole parameter for your Auto Scaling group, instead of a JSON file\.

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
+ The percentage of the group to launch as On\-Demand Instances \(`0`\) and a base number of On\-Demand Instances to start with \(`1`\)\.
+ The instance types to launch in priority order \(`c5.large`, `c5a.large`, `m5.large`, `m5a.large`, `c4.large`, `m4.large`, `c3.large`, `m3.large`\) \.
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\)\. Each corresponds to a different Availability Zone\.
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Latest`\)\.

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c5.large` instance type first\. When Amazon EC2 Auto Scaling attempts to fulfill your Spot capacity, it honors the instance type priorities on a best\-effort basis\. However, it optimizes for capacity first\.

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

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group\. This references a YAML file as the sole parameter for your Auto Scaling group, instead of a JSON file\. 

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
+ The percentage of the group to launch as On\-Demand Instances \(`50`\)\. \(This does not specify a base number of On\-Demand Instances to start with\.\)
+ The instance types to launch in priority order \(`c5.large`, `c5a.large`, `m5.large`, `m5a.large`, `c4.large`, `m4.large`, `c3.large`, `m3.large`\)\. 
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\)\. Each corresponds to a different Availability Zone\.
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Latest`\)\.

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c5.large` instance type first\. For your Spot capacity, Amazon EC2 Auto Scaling attempts to launch the Spot Instances evenly across the two lowest priced pools in each Availability Zone\. 

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

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group\. This references a YAML file as the sole parameter for your Auto Scaling group, instead of a JSON file\. 

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

### Example 4: Launch Spot Instances using the `price-capacity-optimized` allocation strategy<a name="price-capacity-optimized-aws-cli"></a>

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following:
+ The percentage of the group to launch as On\-Demand Instances \(`30`\)\. \(This does not specify a base number of On\-Demand Instances to start with\.\)
+ The instance types to launch in priority order \(`c5.large`, `c5a.large`, `m5.large`, `m5a.large`, `c4.large`, `m4.large`, `c3.large`, `m3.large`\)\. 
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\)\. Each corresponds to a different Availability Zone\.
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Latest`\)\.

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c5.large` instance type first\. For your Spot capacity, Amazon EC2 Auto Scaling attempts to launch the Spot Instances from Spot Instances pools with the lowest price possible, but also with optimal capacity for the number of instances that are launching

#### JSON<a name="price-capacity-optimized-aws-cli-json"></a>

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
            "OnDemandPercentageAboveBaseCapacity": 30,
            "SpotAllocationStrategy": "price-capacity-optimized"
        }
    },
    "MinSize": 1,
    "MaxSize": 5,
    "DesiredCapacity": 3,
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782"
}
```

#### YAML<a name="price-capacity-optimized-aws-cli-yaml"></a>

Alternatively, you can use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create the Auto Scaling group\. This references a YAML file as the sole parameter for your Auto Scaling group, instead of a JSON file\. 

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
    OnDemandPercentageAboveBaseCapacity: 30
    SpotAllocationStrategy: price-capacity-optimized
MinSize: 1
MaxSize: 5
DesiredCapacity: 3
VPCZoneIdentifier: subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782
```

**Note**  
For additional examples, see [Specify a different launch template for an instance type](ec2-auto-scaling-mixed-instances-groups-launch-template-overrides.md), [Configure instance weighting for Amazon EC2 Auto Scaling](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md), and [Use Capacity Rebalancing to handle Amazon EC2 Spot interruptions](ec2-auto-scaling-capacity-rebalancing.md)\. 

## Verify your Auto Scaling group configuration and launched instances \(AWS CLI\)<a name="verify-launch-aws-cli"></a>

To check whether your Auto Scaling group is configured correctly and that it has launched instances, use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\. Verify that the mixed instances policy and the list of subnets exist and are configured correctly\. If the instances have launched, you see a list of the instances and their statuses\. To view the scaling activities that result from instances launching, use the [describe\-scaling\-activities](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command\. You can monitor scaling activities that are in progress and that are recently completed\. For more information, see [Verify a scaling activity for an Auto Scaling group](as-verify-scaling-activity.md)\.