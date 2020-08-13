# Auto Scaling groups with multiple instance types and purchase options<a name="asg-purchase-options"></a>

You can launch and automatically scale a fleet of On\-Demand Instances and Spot Instances within a single Auto Scaling group\. In addition to receiving discounts for using Spot Instances, you can use Reserved Instances or a Savings Plan to receive discounted rates of the regular On\-Demand Instance pricing\. All of these factors combined help you to optimize your cost savings for Amazon EC2 instances, while making sure that you obtain the desired scale and performance for your application\.

You first specify the common configuration parameters in a launch template, and choose it when you create an Auto Scaling group\. When you configure the Auto Scaling group, you can:
+ Choose one or more instance types for the group \(optionally overriding the instance type that is specified by the launch template\)\.
+ Assign each instance type an individual weight\. Doing so might be useful, for example, if the instance types offer different vCPU, memory, storage, or network bandwidth capabilities\.
+ Specify how much On\-Demand and Spot capacity to launch, and specify an optional On\-Demand base portion\.
+ Prioritize instance types that can benefit from Reserved Instance or Savings Plan discount pricing\.
+ Define how Amazon EC2 Auto Scaling should distribute your Spot capacity across instance types\.

You enhance availability by deploying your application across multiple instance types running in multiple Availability Zones\. You can use just one instance type, but it is a best practice to use a few instance types to allow Amazon EC2 Auto Scaling to launch another instance type in the event that there is insufficient instance capacity in your chosen Availability Zones\. With Spot Instances, if there is insufficient instance capacity, Amazon EC2 Auto Scaling keeps trying in other Spot Instance pools \(determined by your choice of instance types and allocation strategy\) rather than launching On\-Demand Instances, so that you can leverage the cost savings of Spot Instances\.

## Allocation strategies<a name="asg-allocation-strategies"></a>

The following allocation strategies determine how the Auto Scaling group fulfills On\-Demand and Spot capacity from the possible instance types\. 

In each case, Amazon EC2 Auto Scaling first tries to ensure that your instances are evenly balanced across the Availability Zones that you specified\. Then, it launches instance types according to the allocation strategy that is specified\. 

### On\-Demand Instances<a name="asg-od-strategy"></a>

The allocation strategy for On\-Demand Instances is `prioritized`\. Amazon EC2 Auto Scaling uses the order of instance types in the list of launch template overrides to determine which instance type to use first when fulfilling On\-Demand capacity\. For example, let's say that you specified three launch template overrides in the following order: `c5.large`, `c4.large`, and `c3.large`\. When your On\-Demand Instances are launched, the Auto Scaling group fulfills On\-Demand capacity by starting with `c5.large`, then `c4.large`, and then `c3.large`\. 

Consider the following when managing the priority order of your On\-Demand Instances:

You can pay for usage upfront to get significant discounts for On\-Demand Instances by using either Reserved Instances or Savings Plans\. For more information about Reserved Instances or Savings Plans, see the [Amazon EC2 pricing](https://aws.amazon.com/ec2/pricing/) page\. 
+ With Reserved Instances, your discounted rate of the regular On\-Demand Instance pricing applies if Amazon EC2 Auto Scaling launches matching instance types\. That means that if you have unused Reserved Instances for `c4.large`, you can set the instance type priority to give the highest priority for your Reserved Instances to a `c4.large` instance type\. When a `c4.large` instance launches, you receive the Reserved Instance pricing\. 
+ With Savings Plans, your discounted rate of the regular On\-Demand Instance pricing applies when using either Amazon EC2 Instance Savings Plans or Compute Savings Plans\. Because of the flexible nature of Savings Plans, you have greater flexibility in prioritizing your instance types\. As long as you use instance types that are covered by your Savings Plan, you can set them in any order of priority and even occasionally change their order entirely, and continue to receive the discounted rate provided by your Savings Plan\. To learn more about Savings Plans, see the [Savings Plans User Guide](https://docs.aws.amazon.com/savingsplans/latest/userguide/)\.

### Spot Instances<a name="asg-spot-strategy"></a>

Amazon EC2 Auto Scaling provides two types of allocation strategies that can be used for Spot Instances: 

`capacity-optimized`  
Amazon EC2 Auto Scaling allocates your instances from the Spot Instance pool with optimal capacity for the number of instances that are launching\. Deploying in this way helps you make the most efficient use of spare EC2 capacity\.   
With Spot Instances, the pricing changes slowly over time based on long\-term trends in supply and demand, but capacity fluctuates in real time\. The `capacity-optimized` strategy automatically launches Spot Instances into the most available pools by looking at real\-time capacity data and predicting which are the most available\. This works well for workloads such as big data and analytics, image and media rendering, and machine learning\. It also works well for high performance computing that may have a higher cost of interruption associated with restarting work and checkpointing\. By offering the possibility of fewer interruptions, the `capacity-optimized` strategy can lower the overall cost of your workload\.

`lowest-price`  
Amazon EC2 Auto Scaling allocates your instances from the number \(N\) of Spot Instance pools that you specify and from the pools with the lowest price per unit at the time of fulfillment\.   
For example, if you specify 4 instance types and 4 Availability Zones, your Auto Scaling group can potentially draw from as many as 16 different Spot Instance pools\. If you specify 2 Spot pools \(N=2\) for the allocation strategy, your Auto Scaling group can fulfill Spot capacity from a minimum of 8 different Spot pools where the price per unit is the lowest\. 

To get started, we recommend choosing the `capacity-optimized` allocation strategy and specifying a few instance types that are appropriate for your application\. In addition, you can define a range of Availability Zones for Amazon EC2 Auto Scaling to choose from when launching instances\. 

Optionally, you can specify a maximum price for your Spot Instances\. If you don't specify a maximum price, the default maximum price is the On\-Demand price, but you still receive the steep discounts provided by Spot Instances\. These discounts are possible because of the stable Spot pricing that is made available using the new [Spot pricing model](https://aws.amazon.com/blogs/compute/new-amazon-ec2-spot-pricing/)\.

For more information about the allocation strategies for Spot Instances, see [Introducing the capacity\-optimized allocation strategy for Amazon EC2 Spot Instances](https://aws.amazon.com/blogs/compute/introducing-the-capacity-optimized-allocation-strategy-for-amazon-ec2-spot-instances/) in the AWS blog\.

## Controlling the proportion of On\-Demand instances<a name="asg-instances-distribution"></a>

You have full control over the proportion of instances in the Auto Scaling group that are launched as On\-Demand Instances\. To ensure that you always have instance capacity, you can designate a percentage of the group to launch as On\-Demand Instances and, optionally, a base number of On\-Demand Instances to start with\. If you choose to specify a base capacity of On\-Demand Instances, the Auto Scaling group ensures that this base capacity of On\-Demand Instances is launched first when the group scales out\. Anything beyond the base capacity uses the On\-Demand percentage to determine how many On\-Demand Instances and Spot Instances to launch\. You can specify any number from 0 to 100 for the On\-Demand percentage\. 

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

## Best practices for Spot Instances<a name="asg-spot-best-practices"></a>

Before you create your Auto Scaling group to request Spot Instances, review [Spot Best Practices](https://aws.amazon.com/ec2/spot/getting-started/#Spot_Best_Practices)\. Use these best practices when you plan your request so that you can provision the type of instances that you want at the lowest possible price\. We also recommend that you do the following: 
+ Use the default maximum price, which is the On\-Demand price\. You pay only the Spot price for the Spot Instances that you launch\. If the Spot price is within your maximum price, whether your request is fulfilled depends on availability\. For more information, see [Pricing and savings](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html#spot-pricing) in the *Amazon EC2 User Guide for Linux Instances*\. 
+ Create your Auto Scaling group with multiple instance types\. Because capacity fluctuates independently for each instance type in an Availability Zone, you can often get more compute capacity when you have instance type flexibility\.
+ Similarly, don't limit yourself to only the most popular instance types\. Because prices adjust based on long\-term demand, popular instance types \(such as recently launched instance families\), tend to have more price adjustments\. Picking older\-generation instance types that are less popular tends to result in lower costs and fewer interruptions\.
+ If you chose the `lowest-price` allocation strategy and you run a web service, specify a high number of Spot pools, for example, N=10\. Specifying a high number of Spot pools reduces the impact of Spot Instance interruptions if a pool in one of the Availability Zones becomes temporarily unavailable\. If you run batch processing or other non\-mission critical applications, you can specify a lower number of Spot pools, for example, N=2\. This helps to ensure that you provision Spot Instances from only the very lowest priced Spot pools available per Availability Zone\. 
+ Use Spot Instance interruption notices to monitor the status of your Spot Instances\. For example, you can set up a rule in Amazon EventBridge that automatically sends the EC2 Spot two\-minute warning to an Amazon SNS topic, an AWS Lambda function, or another target\. For more information, see [Spot Instance interruption notices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#spot-instance-termination-notices) in the *Amazon EC2 User Guide for Linux Instances* and the [Amazon EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/)\.

If you intend to specify a maximum price, use the AWS CLI or AWS SDKs to create the Auto Scaling group, but be cautious\. If your maximum price is lower than the Spot price for the instance types that you selected, your Spot Instances are not launched\. 

## Prerequisites<a name="asg-prerequisites"></a>

Your launch template is configured for use with an Auto Scaling group\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.

IAM users can create an Auto Scaling group using a launch template only if they have permissions to call the `ec2:RunInstances` action\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

## Creating an Auto Scaling group \(console\)<a name="create-asg-multiple-purchase-options"></a>

Follow these steps to create a fleet of On\-Demand Instances and Spot Instances that you can scale\.

Amazon EC2 Auto Scaling has changed the user interface\. By default, you're shown the new user interface, but you can choose to return to the old user interface\. This topic contains steps for each\. 

**To create an Auto Scaling group with multiple purchase options \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch template\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, do the following:

   1. For **Auto Scaling group name**, enter a name for your Auto Scaling group\.

   1. For **Launch template**, choose an existing launch template\.

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\. 

   1. Verify that your launch template supports all of the options that you are planning to use, and then choose **Next**\.

1. On the **Configure settings** page, for **Purchase options and instance types**, choose **Combine purchase options and instance types**\.

1. Under **Instances distribution**, do the following: 

   You can skip these steps if you want to keep the default settings\. 

   1. For **Optional On\-Demand base**, specify the minimum number of instances for the Auto Scaling group's initial capacity that must be fulfilled by On\-Demand Instances\. 

   1. For **On\-Demand percentage above base**, specify the percentages of On\-Demand Instances and Spot Instances for your additional capacity beyond the optional On\-Demand base amount\. 

   1. For **Spot allocation strategy per Availability Zone**, we recommend that you keep the default setting of **Capacity optimized**\. If you prefer not to keep the default, choose **Lowest price**, and then enter the number of lowest priced Spot Instance pools to diversify across\.

1. For **Instance types**, choose which types of instances can be launched, using our recommendations as a starting point\. Otherwise, you can delete the instance types and add them later as needed\. 

1. \(Optional\) To change the order of the instance types, use the arrows\. The order in which you set the instance types sets their priority for On\-Demand Instances\. The instance type at the top of the list is prioritized the highest when the Auto Scaling group launches your On\-Demand capacity\.

1. \(Optional\) To use [instance weighting](asg-instance-weighting.md), assign each instance type a relative weight that corresponds to how much the instance should count toward the capacity of the Auto Scaling group\.

1. Under **Network**, for **VPC**, choose the VPC for the security groups that you specified in your launch template\. Launching instances using multiple instance types and purchase options is not supported in EC2\-Classic\. 

1. For **Subnet**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information about high availability with Amazon EC2 Auto Scaling, see [Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. Choose **Next**\. 

   Or, you can accept the rest of the defaults, and choose **Skip to review**\. 

1. On the **Configure advanced options** page, configure the following options, and then choose **Next**:

   1. \(Optional\) To register your Amazon EC2 instances with an Elastic Load Balancing \(`ELB`\) load balancer, choose **Enable load balancing**\. To attach an Application Load Balancer or Network Load Balancer, choose an existing target group or create a new one\. To attach a Classic Load Balancer, choose an existing load balancer or create a new one\. 

   1. \(Optional\) To enable your `ELB` health checks, for **Health checks**, choose **ELB** under **Health check type**\. 

   1. \(Optional\) Under **Health check grace period**, enter the amount of time until Amazon EC2 Auto Scaling checks the health of instances after they are put into service\. The intention is to prevent Amazon EC2 Auto Scaling from marking instances as unhealthy and terminating them before they have time to come up\. The default is 300 seconds\.

1. On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. \(Optional\) For **Desired capacity**, enter the initial number of instances to launch\. When you change this number to a value outside of the minimum or maximum capacity limits, you must update the values of **Minimum capacity** or **Maximum capacity**\. For more information, see [Setting capacity limits for your Auto Scaling group](asg-capacity-limits.md)\. 

   1. \(Optional\) To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target Tracking Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

   1. \(Optional\) Under **Instance scale\-in protection**, choose whether to enable instance scale\-in protection\. For more information, see [Instance scale\-in protection](as-instance-termination.md#instance-protection)\.

1. \(Optional\) To receive notifications, for **Add notification**, configure the notification, and then choose **Next**\. For more information, see [Getting Amazon SNS notifications when your Auto Scaling group scales](ASGettingNotifications.md)\.

1. \(Optional\) To add tags, choose **Add tag**, provide a tag key and value for each tag, and then choose **Next**\. For more information, see [Tagging Auto Scaling groups and instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

**To create an Auto Scaling group with multiple purchase options \(old console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch template\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1. Choose **Launch Template**, choose your launch template, and then choose **Next Step**\.

1. On the **Configure Auto Scaling group details** page, for **Group name**, enter a name for your Auto Scaling group\.

1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

1. For **Fleet Composition**, choose **Combine purchase options and instances** to launch instances across multiple instance types using both On\-Demand and Spot purchase options\.

1. For **Instance Types**, do the following:

   1. Choose the instance types to use as overrides\. By default, a single instance type is specified by the chosen launch template, but it can be changed if you want to specify a different instance type\. When choosing multiple instance types, the order in which you add instance types sets their priority for On\-Demand Instances\. The instance type at the top of the list is prioritized the highest when the Auto Scaling group launches your On\-Demand capacity\. 

   1. \(Optional\) To use [instance weighting](asg-instance-weighting.md), assign each instance type a relative weight that corresponds with how much the instance should count toward the desired capacity of the Auto Scaling group\.

1. For **Instances Distribution**, choose whether to keep or edit the default settings\. Clearing the box for **Use the default settings to get started quickly** allows you to edit the default settings\. 

1. If you chose to edit the default settings, provide the following information\. 
   + For **Maximum Spot Price**, choose **Use default** to cap your maximum Spot price at the On\-Demand price\. Or choose **Set your maximum price** and enter a value to specify the maximum price you are willing to pay per instance per hour for Spot Instances\.
   + For **Spot Allocation Strategy**, the default is **Launch Spot Instances optimally based on the available Spot capacity per Availability Zone**\. To use the lowest price strategy instead, choose **Diversify Spot Instances across your N lowest priced instance types per Availability Zone**, and then enter the number of Spot pools to use, or accept the default \(2\)\.
   + For **Optional On\-Demand Base**, specify the minimum number of instances for the Auto Scaling group's initial capacity that must be fulfilled by On\-Demand Instances\.
   + For **On\-Demand Percentage Above Base**, specify the percentages of On\-Demand Instances and Spot Instances for your additional capacity beyond the optional On\-Demand base amount\. 

1. For **Group size**, enter the initial number of instances for your Auto Scaling group\.

1. For **Network**, choose a VPC for your Auto Scaling group\. Launching instances using multiple instance types and purchase options is not supported in EC2\-Classic\. 

1. For **Subnet**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information about high availability with Amazon EC2 Auto Scaling, see [Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. \(Optional\) To register your Amazon EC2 instances with a load balancer, choose **Advanced Details**, choose **Receive traffic from one or more load balancers**, and choose one or more Classic Load Balancers or target groups\.

1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, choose one of the following options, and then choose **Next: Configure Notifications**:
   + To manually adjust the size of the Auto Scaling group as needed, choose **Keep this group at its initial size**\. For more information, see [Manual scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.
   + To automatically adjust the size of the Auto Scaling group based on criteria that you specify, choose **Use scaling policies to adjust the capacity of this group** and follow the directions\. For more information, see [Configure Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

1. \(Optional\) To receive notifications, choose **Add notification**, configure the notification, and then choose **Next: Configure Tags**\.

1. \(Optional\) To add tags, choose **Edit tags**, provide a tag key and value for each tag, and then choose **Review**\.

   Alternatively, you can add tags later on\. For more information, see [Tagging Auto Scaling groups and instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

## Creating an Auto Scaling group \(AWS CLI\)<a name="create-asg-multiple-purchase-options-aws-cli"></a>

The following examples show how to create an Auto Scaling group with multiple purchase options using the AWS CLI [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command\.

**Example: Launch Spot Instances using the `capacity-optimized` allocation strategy**

The following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following:
+ The percentage of the group to launch as On\-Demand Instances \(`0`\) and a base number of On\-Demand Instances to start with \(`1`\)
+ The instance types to launch in priority order \(`c3.large`, `c4.large`, `c5.large`\) 
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\), each corresponding to a different Availability Zone
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Default`\)

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c3.large` instance type first\. The Spot Instances come from the optimal Spot pool in each Availability Zone based on Spot Instance capacity\.

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
                    "InstanceType": "c3.large"
                },
                {
                    "InstanceType": "c4.large"
                },
                {
                    "InstanceType": "c5.large"
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
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
    "Tags": []
}
```

**Example: Launch Spot Instances using the `lowest-price` allocation strategy diversified over two pools**

The following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following:
+ The percentage of the group to launch as On\-Demand Instances \(`50`\) without also specifying a base number of On\-Demand Instances to start with
+ The instance types to launch in priority order \(`c3.large`, `c4.large`, `c5.large`\) 
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\), each corresponding to a different Availability Zone
+ The launch template \(`my-launch-template`\) and the launch template version \(`$Latest`\)

When Amazon EC2 Auto Scaling attempts to fulfill your On\-Demand capacity, it launches the `c3.large` instance type first\. For your Spot capacity, Amazon EC2 Auto Scaling attempts to launch the Spot Instances evenly across the two lowest\-priced pools in each Availability Zone\. 

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
                    "InstanceType": "c3.large"
                },
                {
                    "InstanceType": "c4.large"
                },
                {
                    "InstanceType": "c5.large"
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
    "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
    "Tags": []
}
```

**To verify that the group has launched instances**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names my-asg
```

The following example response shows that the desired capacity is `3` and that the group has three running instances\. 

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupARN": "arn",
            "ServiceLinkedRoleARN": "arn",
            "TargetGroupARNs": [],
            "SuspendedProcesses": [],
            "DesiredCapacity": 3,
            "MixedInstancesPolicy": {
                "InstancesDistribution": {
                    "SpotAllocationStrategy": "lowest-price",
                    "OnDemandPercentageAboveBaseCapacity": 50,
                    "OnDemandAllocationStrategy": "prioritized",
                    "SpotInstancePools": 2,
                    "OnDemandBaseCapacity": 0
                },
                "LaunchTemplate": {
                    "LaunchTemplateSpecification": {
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "$Latest",
                        "LaunchTemplateId": "lt-050555ad16a3f9c7f"
                    },
                    "Overrides": [
                        {
                            "InstanceType": "c3.large"
                        },
                        {
                            "InstanceType": "c4.large"
                        },
                        {
                            "InstanceType": "c5.large"
                        }
                    ]
                }
            },
            "EnabledMetrics": [],
            "Tags": [],
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
                    "InstanceId": "i-0aae8709d49eeba4f",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService"
                },
                {
                    "ProtectedFromScaleIn": false,
                    "AvailabilityZone": "us-west-2b",
                    "LaunchTemplate": {
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "1",
                        "LaunchTemplateId": "lt-050555ad16a3f9c7f"
                    },
                    "InstanceId": "i-0c43f6003841d2d2b",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService"
                },
                {
                    "ProtectedFromScaleIn": false,
                    "AvailabilityZone": "us-west-2c",
                    "LaunchTemplate": {
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "1",
                        "LaunchTemplateId": "lt-050555ad16a3f9c7f"
                    },
                    "InstanceId": "i-0feb4cd6677d39903",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService"
                }
            ],
            "MaxSize": 5,
            "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
            "HealthCheckGracePeriod": 0,
            "TerminationPolicies": [
                "Default"
            ],
            "LoadBalancerNames": [],
            "CreatedTime": "2019-02-17T02:29:12.853Z",
            "AvailabilityZones": [
                "us-west-2a",
                "us-west-2b",
                "us-west-2c"
            ],
            "HealthCheckType": "EC2",
            "NewInstancesProtectedFromScaleIn": false
        }
    ]
}
```

For additional examples, see [Instance weighting for Amazon EC2 Auto Scaling](asg-instance-weighting.md)\. 