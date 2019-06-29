# Auto Scaling Groups with Multiple Instance Types and Purchase Options<a name="asg-purchase-options"></a>

You can launch and automatically scale a fleet of On\-Demand Instances and Spot Instances within a single Auto Scaling group\. In addition to receiving discounts for using Spot Instances, if you specify instance types for which you have matching Reserved Instances, your discounted rate of the regular On\-Demand Instance pricing also applies\. The only difference between On\-Demand Instances and Reserved Instances is that you must purchase the Reserved Instances in advance\. All of these factors combined help you to optimize your cost savings for Amazon EC2 instances, while making sure that you obtain the desired scale and performance for your application\. 

You first specify the common configuration parameters in a launch template and choose it when you create an Auto Scaling group\. When you configure the Auto Scaling group, you can specify how much On\-Demand and Spot capacity to launch, choose the instance types that work best for your application, and define how Amazon EC2 Auto Scaling should distribute your capacity across instance types within each purchasing option\. 

You enhance availability by deploying your application across multiple instance types running in multiple Availability Zones\. Deploying in this way also gives you the benefit of the least expensive and available instance types in the Spot market\. You must specify a minimum of two instance types, but it is a best practice to choose a few instance types to avoid trying to launch instances from instance pools with insufficient capacity\. If the Auto Scaling group's request for Spot Instances cannot be fulfilled in one Spot Instance pool, it keeps trying in other Spot Instance pools rather than launching On\-Demand Instances, so that you can leverage the cost savings of Spot Instances\. 

## Allocation Strategies<a name="asg-allocation-strategies"></a>

The following allocation strategies determine how the Auto Scaling group fulfills On\-Demand and Spot capacity from the possible instance types\. 

### On\-Demand Instances<a name="asg-od-strategy"></a>

The allocation strategy for On\-Demand Instances is `prioritized`\. The Auto Scaling group uses the order of instance types in the list of launch template overrides to determine which instance type to use first when fulfilling On\-Demand capacity\.

For example, you specified three launch template overrides in the following order: `c5.large`, `c4.large`, and `c3.large`\. When your On\-Demand Instances are launched, the Auto Scaling group fulfills On\-Demand capacity by starting with `c5.large`, then `c4.large`, and then `c3.large`\. If you have unused Reserved Instances for `c4.large`, you can set the priority of your instance types to give the highest priority to your Reserved Instances by specifying `c4.large` as the highest priority instance type\. When a `c4.large` instance launches, you receive the Reserved Instance pricing\.

### Spot Instances<a name="asg-spot-strategy"></a>

The allocation strategy for Spot Instances is `lowest-price`\. Spot Instances are distributed across the number of Spot Instance pools that you specify, and come from the pools with the lowest price per unit at the time of fulfillment\. 

A Spot Instance pool is a set of unused EC2 instances with the same instance type, operating system, Availability Zone, and network platform\. The number of instance types and Availability Zones \(both of which are specified in the group\) directly contributes to how many Spot pools Amazon EC2 Auto Scaling can draw from\. For example, if you specify 4 instance types and 4 Availability Zones, your Auto Scaling group can potentially fulfill Spot capacity from as many as 16 different Spot pools\. If you specify 2 Spot pools \(N=2\) for the allocation strategy, your Auto Scaling group can fulfill Spot capacity from a minimum of 8 different Spot pools \(the 2 least expensive Spot pools per Availability Zone\)\.

## Controlling the Proportion of On\-Demand Instances<a name="asg-instances-distribution"></a>

You have full control over the proportion of instances in the Auto Scaling group that are launched as On\-Demand Instances\. To ensure that you always have instance capacity, you can designate a percentage of the group to launch as On\-Demand Instances and, optionally, a base number of On\-Demand Instances to start with\. If you choose to specify a base capacity of On\-Demand Instances, the Auto Scaling group ensures that this base capacity of On\-Demand Instances is launched first when the group scales out\. Anything beyond the base capacity uses the On\-Demand percentage to determine how many On\-Demand Instances and Spot Instances to launch\. You can specify any number from 0 to 100 for the On\-Demand percentage\. 

The behavior of the Auto Scaling group as it increases in size is as follows:


**Example: Scaling behavior**  

| Instances Distribution | Total Number of Running Instances Across Purchase Options | 
| --- | --- | 
|  | 10 | 20 | 30 | 40 | 
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

## Best Practices for Spot Instances<a name="asg-spot-best-practices"></a>

Before you create your Auto Scaling group to request Spot Instances, review [Spot Best Practices](https://aws.amazon.com/ec2/spot/getting-started/#Spot_Best_Practices)\. Use these best practices when you plan your request so that you can provision the type of instances you want at the lowest possible price\. We also recommend that you do the following: 
+ Use the default maximum price, which is the On\-Demand price\. You pay only the Spot market price for the Spot Instances that you launch\. If the Spot market price is within your maximum price, whether your request is fulfilled depends on availability\. For more information, see [Pricing and Savings](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html#spot-pricing) in the *Amazon EC2 User Guide for Linux Instances*\. 
+ Create your Auto Scaling group with multiple instance types\. The minimum is two\. Because prices fluctuate independently for each instance type in an Availability Zone, you can often get more compute capacity for the same price when you have instance type flexibility\.
+ Similarly, don't limit yourself to only the most popular instance types\. Because prices adjust based on long\-term demand, popular instance types \(such as recently launched instance families\), tend to have more price adjustments\. Picking older\-generation instance types that are less popular tends to result in lower costs and fewer interruptions\.
+ If you run a web service, specify a high number of Spot pools, for example, N=10, to reduce the impact of Spot Instance interruptions if a pool in one of the Availability Zones becomes temporarily unavailable\. If you run batch processing or other non\-mission critical applications, you can specify a lower number of Spot pools, for example, N=2\. This helps to ensure that you provision Spot Instances from only the very lowest priced Spot pools available per Availability Zone\. 
+ Use Spot Instance interruption notices to monitor the status of your Spot Instances\. For example, you can set up a rule in Amazon CloudWatch Events that automatically sends the EC2 Spot two\-minute warning to an Amazon SNS topic, an AWS Lambda function, or another target\. For more information, see [Spot Instance Interruption Notices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#spot-instance-termination-notices) in the *Amazon EC2 User Guide for Linux Instances* and the [Amazon CloudWatch Events User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)\.

## Prerequisites<a name="asg-prerequisites"></a>

Your launch template is configured for use with an Auto Scaling group\. For information, see [Creating a Launch Template for an Auto Scaling Group](create-launch-template.md)\.

IAM users can create an Auto Scaling group using a launch template only if they have permissions to call the `ec2:RunInstances` action\. For information about configuring user permissions, see [Controlling Access to Your Amazon EC2 Auto Scaling Resources](control-access-using-iam.md)\.

## Creating an Auto Scaling Group with Multiple Purchase Options<a name="create-asg-multiple-purchase-options"></a>

Follow these steps to create a fleet of On\-Demand Instances and Spot Instances that you can scale\. You can quickly create your fleet by using the default settings for **Instances Distribution**\. Otherwise, you can modify any of the default settings\. You have full control over scaling the group\. You can set desired capacity to a specific number of instances, or you can take advantage of automatic scaling to adjust capacity based on actual demand\. 

**To create an Auto Scaling group with multiple purchase options \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, choose the same AWS Region that you used when you created the launch template\.

1. In the navigation pane, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1. Choose **Launch Template**, choose your launch template, and then choose **Next Step**\.

1. On the **Configure Auto Scaling group details** page, for **Group name**, enter a name for your Auto Scaling group\.

1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

1. For **Fleet Composition**, choose **Combine purchase options and instances** to launch instances across multiple instance types using both On\-Demand and Spot purchase options\.

1. For **Instance Types**, choose the optimal instance types \(such as `c5.large`\) that can be launched\. The order in which you add instance types sets their priority for On\-Demand Instances\. The instance type at the top of the list is prioritized the highest when the Auto Scaling group launches your On\-Demand capacity\. You must specify at least two instance types\.

1. For **Instances Distribution**, choose whether to keep or edit the default settings\. Clearing the box for **Use the default settings to get started quickly** allows you to edit the default settings\. 

1. If you chose to edit the default settings, provide the following information\. 
   + For **Maximum Spot Price**, choose **Use default** to cap your maximum Spot price at the On\-Demand price\. Or choose **Set your maximum price** and enter a value to specify the maximum price you are willing to pay per instance per hour for Spot Instances\.
   + For **Spot Allocation Strategy**, choose the number of Spot pools to allocate your Spot Instances across\. 
   + For **Optional On\-Demand Base**, specify the minimum number of instances for the Auto Scaling group's initial capacity that must be fulfilled by On\-Demand Instances\.
   + For **On\-Demand Percentage Above Base**, specify the percentages of On\-Demand Instances and Spot Instances for your additional capacity beyond the optional On\-Demand base amount\. 

1. For **Group size**, enter the initial number of instances for your Auto Scaling group\.

1. For **Network**, choose a VPC for your Auto Scaling group\.
**Note**  
Launching instances using a combination of instance types and purchase options is not supported in EC2\-Classic\. 

1. For **Subnet**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information about high availability with Amazon EC2 Auto Scaling, see [Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. \(Optional\) To register your Amazon EC2 instances with a load balancer, choose **Advanced Details**, choose **Receive traffic from one or more load balancers**, and choose one or more Classic Load Balancers or target groups\.

1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, choose one of the following options, and then choose **Next: Configure Notifications**:
   + To manually adjust the size of the Auto Scaling group as needed, choose **Keep this group at its initial size**\. For more information, see [Manual Scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.
   + To automatically adjust the size of the Auto Scaling group based on criteria that you specify, choose **Use scaling policies to adjust the capacity of this group** and follow the directions\. For more information, see [Configure Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

1. \(Optional\) To receive notifications, choose **Add notification**, configure the notification, and then choose **Next: Configure Tags**\.

1. \(Optional\) To add tags, choose **Edit tags**, provide a tag key and value for each tag, and then choose **Review**\.

   Alternatively, you can add tags later on\. For more information, see [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

**To create an Auto Scaling group with multiple purchase options \(AWS CLI\)**  
The following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group that specifies the following: 
+ The instance types to launch in priority order \( `c5.large`, `c4.large`, `c3.large`\) 
+ The number of Spot Instance pools \(`2`\) to allocate Spot Instances across
+ The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\) 

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
                "LaunchTemplateName": "my-template-for-auto-scaling",
                "Version": "1"
            },
            "Overrides": [
                {
                    "InstanceType": "c5.large"
                },
                {
                    "InstanceType": "c4.large"
                },
                {
                    "InstanceType": "c3.large"
                }
            ]
        },
        "InstancesDistribution": {
            "OnDemandBaseCapacity": 1,
            "OnDemandPercentageAboveBaseCapacity": 50,
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
                    "OnDemandBaseCapacity": 1
                },
                "LaunchTemplate": {
                    "LaunchTemplateSpecification": {
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "1",
                        "LaunchTemplateId": "lt-050555ad16a3f9c7f"
                    },
                    "Overrides": [
                        {
                            "InstanceType": "c5.large"
                        },
                        {
                            "InstanceType": "c4.large"
                        },
                        {
                            "InstanceType": "c3.large"
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
                        "LaunchTemplateName": "my-template-for-auto-scaling",
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
                        "LaunchTemplateName": "my-template-for-auto-scaling",
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
                        "LaunchTemplateName": "my-template-for-auto-scaling",
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