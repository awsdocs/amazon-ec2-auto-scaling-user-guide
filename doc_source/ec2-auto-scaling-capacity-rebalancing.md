# Use Capacity Rebalancing to handle Amazon EC2 Spot Interruptions<a name="ec2-auto-scaling-capacity-rebalancing"></a>

You can configure Amazon EC2 Auto Scaling to monitor and automatically respond to changes that affect the availability of your Spot Instances\. Capacity Rebalancing helps you maintain workload availability by proactively augmenting your fleet with a new Spot Instance before a running instance is interrupted by Amazon EC2\.

## How it works<a name="capacity-rebalancing-how-it-works"></a>

The goal of Capacity Rebalancing is to keep processing your workload without interruption\. When Spot Instances are at an elevated risk of interruption, the Amazon EC2 Spot service notifies Amazon EC2 Auto Scaling with an EC2 instance rebalance recommendation\.

When you enable Capacity Rebalancing for your Auto Scaling group, Amazon EC2 Auto Scaling attempts to proactively replace the Spot Instances in your group that have received a rebalance recommendation\. This provides an opportunity to rebalance your workload to new Spot Instances that aren't at an elevated risk of interruption\. Your workload can continue to process the work while Amazon EC2 Auto Scaling launches new Spot Instances before your existing instances are interrupted\.

Optionally, you can also use a lifecycle hook to perform a custom action on the instances before they are terminated\.

For more information about EC2 instance rebalance recommendations, see [EC2 instance rebalance recommendations](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/rebalance-recommendations.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

For more details and a walkthrough of the Capacity Rebalancing feature, see the blog post [Proactively manage Spot Instance lifecycle using the new Capacity Rebalancing feature for EC2 Auto Scaling](http://aws.amazon.com/blogs/compute/proactively-manage-spot-instance-lifecycle-using-the-new-capacity-rebalancing-feature-for-ec2-auto-scaling) on the AWS Compute Blog\. 

**Note**  
When Capacity Rebalancing is disabled, Amazon EC2 Auto Scaling doesn't replace Spot Instances until after the Amazon EC2 Spot service interrupts the instances and their health check fails\. Before interrupting an instance, Amazon EC2 always gives both an EC2 instance rebalance recommendation and a Spot two\-minute instance interruption notice\. 

**Contents**
+ [How it works](#capacity-rebalancing-how-it-works)
+ [Enable Capacity Rebalancing](#enable-capacity-rebalancing)
  + [Enable Capacity Rebalancing \(console\)](#enable-capacity-rebalancing-console)
  + [Enable Capacity Rebalancing \(AWS CLI\)](#enable-capacity-rebalancing-aws-cli)
+ [Add a termination lifecycle hook](#capacity-rebalancing-lifecycle-hook)

## Enable Capacity Rebalancing<a name="enable-capacity-rebalancing"></a>

You can enable or disable Capacity Rebalancing at any time\. 

**Considerations**

The following considerations apply for this configuration:
+ We recommend that you configure your Auto Scaling group to use multiple instance types\. This provides the flexibility to launch instances in various Spot Instance pools within each Availability Zone, as documented in [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\. 
+ Your replacement Spot Instances may be at an elevated risk of interruption if you use the `lowest-price` allocation strategy\. This is because we will always launch instances in the lowest priced pool that has available capacity at that moment, even if your replacement Spot Instances are likely to be interrupted soon after being launched\. To avoid an elevated risk of interruption, we strongly recommend against using the `lowest-price` allocation strategy, and instead recommend the `capacity-optimized` or `capacity-optimized-prioritized` allocation strategy\. These strategies aim to launch replacement Spot Instances in the most optimal Spot capacity pools so that they are less likely to be interrupted in the near future\.
+ Whenever possible, you should create your Auto Scaling group in all Availability Zones within the Region\. This way, Amazon EC2 Auto Scaling can look at the available capacity in each Availability Zone\. If a launch fails in one Availability Zone, Amazon EC2 Auto Scaling keeps trying to launch Spot Instances across the specified Availability Zones until it succeeds\. 
+ With Capacity Rebalancing, Amazon EC2 Auto Scaling behaves in the following way: 

  When launching a new instance, Amazon EC2 Auto Scaling waits until the new instance passes its health check before it proceeds with terminating the previous instance\. When replacing more than one instance, the termination of each previous instance starts after the new instance has launched and passed its health check\. Because Amazon EC2 Auto Scaling attempts to launch new instances before terminating previous ones, being at or near the specified maximum capacity could impede or completely halt rebalancing activities\. To avoid this problem, Amazon EC2 Auto Scaling can temporarily exceed the specified maximum capacity of a group during a rebalancing activity\.
  + If the new instances fail to launch or they launch but the health check fails, Amazon EC2 Auto Scaling keeps trying to relaunch them\. While it is trying to launch new instances, your previous ones will eventually be interrupted and forcibly terminated\. 
  + If a scaling activity is in progress and your Auto Scaling group is below its new desired capacity, Amazon EC2 Auto Scaling scales out first before terminating the previous instances\.
+ You can configure a termination lifecycle hook for your Auto Scaling group when enabling Capacity Rebalancing to attempt a graceful shutdown of your application inside the instances that receive the rebalance notification, before Amazon EC2 Auto Scaling terminates the instances\. If you don't configure a lifecycle hook, Amazon EC2 Auto Scaling starts terminating the previous instances as soon as the new instances pass their health check\.
+ Your application should be able to handle the possibility of a Spot Instance being interrupted early\. For example, when an instance begins termination, Amazon EC2 Auto Scaling waits for the instance to terminate\. If the Auto Scaling group is behind an Elastic Load Balancing load balancer, Amazon EC2 Auto Scaling waits for the instance to deregister from the load balancer before calling the termination lifecycle hook \(if configured\)\. If the time to deregister the instance and complete lifecycle actions takes too long, the instance might be interrupted while Amazon EC2 Auto Scaling is waiting for the instance to terminate\.
+ It is also not always possible for Amazon EC2 to send the rebalance recommendation signal before the two\-minute Spot Instance interruption notice\. In some cases, the rebalance recommendation signal can arrive along with the two\-minute interruption notice\.
+ In cases where an instance receives a final two\-minute interruption notice, Amazon EC2 Auto Scaling calls the termination lifecycle hook and attempts to launch a replacement immediately\.

### Enable Capacity Rebalancing \(console\)<a name="enable-capacity-rebalancing-console"></a>

You can enable or disable Capacity Rebalancing when you create or update an Auto Scaling group\.

**To enable Capacity Rebalancing for a new Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Choose **Create Auto Scaling group**\.

1. In step 1, enter a name for the Auto Scaling group, choose a launch template, and then choose **Next** to proceed to the next step\.

1. For **Step 2: Choose instance launch options**, under **Network**, choose the options as desired\.

1. For **Instance type requirements**, choose settings to create a mixed instances group, including the instance types that it can launch, instance purchase options, and allocation strategies for Spot and On\-Demand Instances\. By default, these settings are not configured\. To configure them, you must select **Override launch template**\. For more information about creating a mixed instances group, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.

1. Under the **Allocation strategies** section at the bottom of the page, choose a Spot allocation strategy\. Enable or disable Capacity Rebalancing by selecting or clearing the **Capacity rebalance** check box\. You only see this option if you specify a percentage of the Auto Scaling group to be launched as Spot Instances in the **Instance purchase options** section\.

1. Create the Auto Scaling group\. 

**To enable Capacity Rebalancing for an existing Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Allocation strategies**, **Edit**\.

1. Under the **Allocation strategies** section, do the following:
   + To enable Capacity Rebalancing, select the **Capacity rebalance** check box\.
   + To disable Capacity Rebalancing, clear the **Capacity rebalance** check box\.

1. Choose **Update**\.

### Enable Capacity Rebalancing \(AWS CLI\)<a name="enable-capacity-rebalancing-aws-cli"></a>

The following examples show how to use the AWS CLI to enable and disable Capacity Rebalancing\. 

Use the [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) or [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command with the following parameter: 
+ `--capacity-rebalance` / `--no-capacity-rebalance` — Boolean value that indicates whether Capacity Rebalancing is enabled\.

Before you call the [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command, you need the name of a launch template that is configured for use with an Auto Scaling group\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\. 

**Note**  
The following procedures show how to use a configuration file formatted in JSON or YAML\. If you use AWS CLI version 1, you must specify a JSON\-formatted configuration file\. If you use AWS CLI version 2, you can specify a configuration file formatted in either YAML or JSON\.

#### JSON<a name="enable-capacity-rebalancing-aws-cli-json"></a>

**To create and configure a new Auto Scaling group**
+ Use the following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create a new Auto Scaling group and enable Capacity Rebalancing, referencing a JSON file as the sole parameter for your Auto Scaling group\.

  ```
  aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
  ```

  If you don't already have a CLI configuration file that specifies a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md), create one\.

  Add the following line to the top\-level JSON object in the configuration file\. 

  ```
  {
      "CapacityRebalance": true
  }
  ```

  The following is an example `config.json` file\. 

  ```
  {
      "AutoScalingGroupName": "my-asg",
      "DesiredCapacity": 12,
      "MinSize": 12,
      "MaxSize": 15,
      "CapacityRebalance": true,
      "MixedInstancesPolicy": {
          "InstancesDistribution": {
              "OnDemandBaseCapacity": 0,
              "OnDemandPercentageAboveBaseCapacity": 25,
              "SpotAllocationStrategy": "capacity-optimized"
          },
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
          }
      },
      "TargetGroupARNs": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-alb-target-group/943f017f100becff",
      "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782"
  }
  ```

#### YAML<a name="enable-capacity-rebalancing-aws-cli-yaml"></a>

**To create and configure a new Auto Scaling group**
+ Use the following [create\-auto\-scaling\-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create a new Auto Scaling group and enable Capacity Rebalancing, referencing a YAML file as the sole parameter for your Auto Scaling group\.

  ```
  aws autoscaling create-auto-scaling-group --cli-input-yaml file://~/config.yaml
  ```

  Add the following line to your configuration file formatted in YAML\.

  ```
  CapacityRebalance: true
  ```

  The following is an example `config.yaml` file\. 

  ```
  ---
  AutoScalingGroupName: my-asg
  DesiredCapacity: 12
  MinSize: 12
  MaxSize: 15
  CapacityRebalance: true
  MixedInstancesPolicy:
    InstancesDistribution:
      OnDemandBaseCapacity: 0
      OnDemandPercentageAboveBaseCapacity: 25
      SpotAllocationStrategy: capacity-optimized
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
  TargetGroupARNs:
  - arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-alb-target-group/943f017f100becff
  VPCZoneIdentifier: subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782
  ```

**To enable Capacity Rebalancing for an existing Auto Scaling group**
+ Use the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to enable Capacity Rebalancing\.

  ```
  aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
    --capacity-rebalance
  ```

**To verify that Capacity Rebalancing is enabled for an Auto Scaling group**
+ Use the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that Capacity Rebalancing is enabled and to view the details\. 

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
              ...
              "CapacityRebalance": true
          }
      ]
  }
  ```

**To disable Capacity Rebalancing**  
Use the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command with the `--no-capacity-rebalance` option to disable Capacity Rebalancing\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --no-capacity-rebalance
```

## Add a termination lifecycle hook<a name="capacity-rebalancing-lifecycle-hook"></a>

Optionally, configure a termination lifecycle hook when you enable Capacity Rebalancing\. With a lifecycle hook, you can perform custom actions on an instance before it is terminated\.

The following are some reasons why you might use a termination lifecycle hook: 
+ To upload system or application logs to Amazon Simple Storage Service \(Amazon S3\)
+ For graceful shutdown of Amazon SQS workers
+ To complete deregistration from the Domain Name System \(DNS\)

If you don't have a termination lifecycle hook, use the following procedure to create one\.

**To add a termination lifecycle hook**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens in the bottom of the **Auto Scaling groups** page\. 

1. On the **Instance management** tab, in **Lifecycle hooks**, choose **Create lifecycle hook**\.

1. To define a lifecycle hook, do the following:

   1. For **Lifecycle hook name**, specify a name for the lifecycle hook\.

   1. For **Lifecycle transition**, choose **Instance terminate**\. 

   1. For **Heartbeat timeout**, specify the amount of time, in seconds, that you will have to complete the lifecycle action, or until the timeout period ends\. We recommend a value from 30 to 120 seconds, depending on how much time you need to shut down your application\.

   1. For **Default result**, specify the action that the Auto Scaling group takes when the timeout elapses or if an unexpected failure occurs\. Both **ABANDON** and **CONTINUE** let the instance terminate\. 
      + If you choose **CONTINUE**, the Auto Scaling group can proceed with any remaining actions, such as other lifecycle hooks, before termination\. 
      + If you choose **ABANDON**, the Auto Scaling group terminates the instances immediately\. 

   1. \(Optional\) For **Notification metadata**, specify additional information that you want to include anytime that Amazon EC2 Auto Scaling sends a message to an AWS Lambda function or another notification target that you configure in step 6\. 

1. Choose **Create**\.

1. \(Optional\) To use a service such as Lambda to perform a custom action before instance termination, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\. This tutorial will help you understand how to set up a Lambda function for your lifecycle hook\. Otherwise, for an EC2 instance to run an action automatically, you must configure it to run a shutdown script\. We recommend that you script your entire shutdown sequence to be completed in under one to two minutes to make sure that there is enough time to complete tasks before instance termination\. 

For more information to help you understand different aspects of working with lifecycle hooks, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\. 