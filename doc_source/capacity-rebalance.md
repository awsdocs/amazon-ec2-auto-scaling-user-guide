# Amazon EC2 Auto Scaling Capacity Rebalancing<a name="capacity-rebalance"></a>

You can configure Amazon EC2 Auto Scaling to monitor and automatically respond to changes that affect the availability of your Spot Instances\. Capacity Rebalancing helps you maintain workload availability by proactively augmenting your fleet with a new Spot Instance before a running instance is interrupted by EC2\.

## How it works<a name="auto-scaling-capacity-rebalance-how-it-works"></a>

Amazon EC2 Auto Scaling is aware of EC2 instance rebalance recommendation notifications\. The Amazon EC2 Spot service emits these notifications when Spot Instances are at elevated risk of interruption\. When Capacity Rebalancing is enabled for an Auto Scaling group, Amazon EC2 Auto Scaling attempts to proactively replace Spot Instances in the group that have received a rebalance recommendation, providing the opportunity to rebalance your workload to new Spot Instances that are not at elevated risk of interruption\. This means that your workload can continue to process the work while Amazon EC2 Auto Scaling launches a new Spot Instance before an existing instance is interrupted\. You can also optionally use a lifecycle hook to perform a custom action on instances before termination\.

For more information about EC2 instance rebalance recommendations, see [EC2 instance rebalance recommendations](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/rebalance-recommendations.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

For more details and a walkthrough of the Capacity Rebalancing feature, see the blog post [Proactively manage Spot Instance lifecycle using the new Capacity Rebalancing feature for EC2 Auto Scaling](http://aws.amazon.com/blogs/compute/proactively-manage-spot-instance-lifecycle-using-the-new-capacity-rebalancing-feature-for-ec2-auto-scaling) on the AWS Compute Blog\. 

**Note**  
When Capacity Rebalancing is disabled, Amazon EC2 Auto Scaling replaces Spot Instances after the Amazon EC2 Spot service interrupts the instances and their health check fails\. Amazon EC2 always gives both an EC2 instance rebalance recommendation and a Spot two\-minute instance interruption notice before an instance is interrupted\. 

**Contents**
+ [How it works](#auto-scaling-capacity-rebalance-how-it-works)
+ [Enabling Capacity Rebalancing](#auto-scaling-enabling-capacity-rebalance)
  + [Enabling Capacity Rebalancing \(console\)](#enable-capacity-rebalance-console)
  + [Enabling Capacity Rebalancing \(AWS CLI\)](#enable-capacity-rebalance-aws-cli)
+ [Adding a termination lifecycle hook](#auto-scaling-capacity-rebalance-lifecycle-hook)

## Enabling Capacity Rebalancing<a name="auto-scaling-enabling-capacity-rebalance"></a>

You can enable or disable Capacity Rebalancing at any time\. 

**Considerations**

The following considerations apply for this configuration:
+ We recommend that you configure your Auto Scaling group to use multiple instance types\. This provides the flexibility to launch instances in various Spot Instance pools within each Availability Zone, as documented in [Auto Scaling groups with multiple instance types and purchase options](asg-purchase-options.md)\. 
+ We highly recommend that you only use the Capacity Rebalancing feature with either the `capacity-optimized` or the `capacity-optimized-prioritized` allocation strategy\. These allocation strategies will maintain your Spot capacity in the optimal Spot pools for both scale\-out events and Capacity Rebalancing launches\.
+ Whenever possible, you should create your Auto Scaling group in all Availability Zones within the Region\. This is to enable Amazon EC2 Auto Scaling to look at the free capacity in each Availability Zone\. If a launch fails in one Availability Zone, Amazon EC2 Auto Scaling keeps trying to launch Spot Instances across the specified Availability Zones until it succeeds\. 
+ Using Capacity Rebalancing, Amazon EC2 Auto Scaling behaves in the following way: 

  When launching a new instance, Amazon EC2 Auto Scaling waits until the new instance passes its health check before it proceeds with terminating the old instance\. When replacing more than one instance, the termination of each old instance starts after the new instance has launched and passed its health check\. Because Amazon EC2 Auto Scaling attempts to launch new instances before terminating old ones, being at or near the specified maximum capacity could impede or completely halt rebalancing activities\. To avoid this problem, Amazon EC2 Auto Scaling can temporarily exceed the specified maximum capacity of a group during a rebalancing activity\.
  + If the new instances fail to launch or they launch but the health check fails, Amazon EC2 Auto Scaling keeps trying to relaunch them\. While it is trying to launch new instances, your old ones will eventually be interrupted and forcibly terminated\. 
  + If a scaling activity is in progress and your Auto Scaling group is below its new desired capacity, Amazon EC2 Auto Scaling scales out first before terminating the old instances\.
+ You can configure a termination lifecycle hook for your Auto Scaling group when enabling Capacity Rebalancing to attempt a graceful shut down of your application inside the instances that receive the rebalance notification, before Amazon EC2 Auto Scaling terminates the instances\. If you don't configure a lifecycle hook, Amazon EC2 Auto Scaling starts terminating the old instances as soon as the new instances pass their health check\.
+ When using Capacity Rebalancing, your application should be tolerant of some interruption to prevent disruptions to your application\. When an instance begins termination, Amazon EC2 Auto Scaling waits for the instance to terminate\. If the Auto Scaling group is behind an Elastic Load Balancing load balancer, Amazon EC2 Auto Scaling waits for the instance to deregister from the load balancer before calling the termination lifecycle hook \(if configured\)\. If the time to drain connections and run lifecycle hooks takes too long, the instance may be interrupted while Amazon EC2 Auto Scaling is waiting for the instance to terminate\.
+ In cases where an instance receives a final two\-minute interruption notice, Amazon EC2 Auto Scaling calls the termination lifecycle hook and attempts to launch a replacement immediately\.

### Enabling Capacity Rebalancing \(console\)<a name="enable-capacity-rebalance-console"></a>

You can enable or disable Capacity Rebalancing when you create or update an Auto Scaling group\.

**To enable Capacity Rebalancing for a new Auto Scaling group**  
Follow the instructions in [Auto Scaling groups with multiple instance types and purchase options](asg-purchase-options.md) to create a new Auto Scaling group\. When you create an Auto Scaling group, in step 2 of the wizard, you configure the instance purchase options and network settings\. To specify additional settings for your Auto Scaling group, including instance distribution settings, Spot allocation settings, Capacity Rebalancing, and the instance types that Amazon EC2 Auto Scaling can launch, select **Combine purchase options and instance types**\.

Under the **Instances distribution** section, you can select whether or not to enable Capacity Rebalancing by selecting or clearing the **Capacity rebalance** check box\. This setting is enabled by default on Auto Scaling groups created using the console when the Spot allocation strategy is set to **Capacity optimized**\.

![\[The Capacity rebalance check box\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/asg-console-enable-capacity-rebalance.png)

**To enable Capacity Rebalancing for an existing Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Purchase options and instance types**, **Edit**\.

1. Under the **Instances distribution** section, do the following:
   + To enable Capacity Rebalancing, select the **Capacity rebalance** check box\.
   + To disable Capacity Rebalancing, clear the **Capacity rebalance** check box\.

1. Choose **Update**\.

### Enabling Capacity Rebalancing \(AWS CLI\)<a name="enable-capacity-rebalance-aws-cli"></a>

The following examples show how to use the AWS CLI to enable and disable Capacity Rebalancing\. 

Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) or [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command with the following parameter: 
+ `--capacity-rebalance` / `--no-capacity-rebalance` — Boolean value that indicates whether Capacity Rebalancing is enabled\.

Before you call the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command, you need the name of a launch template that is configured for use with an Auto Scaling group\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\. 

**Note**  
The following procedures show how to use a configuration file formatted in JSON or YAML\. If you use AWS CLI version 1, you must specify a JSON\-formatted configuration file\. If you use AWS CLI version 2, you can specify a configuration file formatted in either YAML or JSON\.

#### JSON<a name="enable-capacity-rebalance-aws-cli-json"></a>

**To create and configure a new Auto Scaling group**
+ Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create a new Auto Scaling group and enable Capacity Rebalancing, referencing a JSON file as the sole parameter for your Auto Scaling group\.

  ```
  aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
  ```

  If you don't already have a CLI configuration file that specifies a [mixed instances policy](asg-purchase-options.md), create one\.

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

#### YAML<a name="enable-capacity-rebalance-aws-cli-yaml"></a>

**To create and configure a new Auto Scaling group**
+ Use the following [https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/autoscaling/create-auto-scaling-group.html) command to create a new Auto Scaling group and enable Capacity Rebalancing, referencing a YAML file as the sole parameter for your Auto Scaling group\.

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
+ Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to enable Capacity Rebalancing\.

  ```
  aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
    --capacity-rebalance
  ```

**To verify that Capacity Rebalancing is enabled for an Auto Scaling group**
+ Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that Capacity Rebalancing is enabled and to view the details\. 

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
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command with the `--no-capacity-rebalance` option to disable Capacity Rebalancing\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --no-capacity-rebalance
```

## Adding a termination lifecycle hook<a name="auto-scaling-capacity-rebalance-lifecycle-hook"></a>

Consider configuring a termination lifecycle hook when enabling Capacity Rebalancing so that you can run code and perform custom actions on an instance before it is terminated\.

Reasons you might use a termination lifecycle hook include: 
+ To upload system or application logs to Amazon Simple Storage Service \(Amazon S3\)
+ For graceful shutdown of Amazon SQS workers
+ To complete deregistration from the Domain Name System \(DNS\)

If you don't have a termination lifecycle hook, use the following procedure to create one\.

**To add a termination lifecycle hook**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Instance management** tab, in **Lifecycle hooks**, choose **Create lifecycle hook**\.

1. To define a lifecycle hook, do the following:

   1. For **Lifecycle hook name**, specify a name for the lifecycle hook\.

   1. For **Lifecycle transition**, choose **Instance terminate**\. 

   1. For **Heartbeat timeout**, specify the amount of time, in seconds, that you will have to complete the lifecycle action, or until the timeout period ends\. We recommend a value from 30 to 120 seconds, depending on how long you need to shut down your application\.

   1. For **Default result**, specify the action that the Auto Scaling group takes when the lifecycle hook timeout elapses or if an unexpected failure occurs\. Both ABANDON and CONTINUE allow the instance to terminate\. 
      + If you choose CONTINUE, the Auto Scaling group can proceed with any remaining actions, such as other lifecycle hooks, before termination\. 
      + If you choose ABANDON, the Auto Scaling group terminates the instances immediately\. 

   1. \(Optional\) For **Notification metadata**, specify additional information that you want to include anytime that Amazon EC2 Auto Scaling sends a message to an AWS Lambda function or another target that you configure in step 6\. 

1. Choose **Create**\.

1. \(Optional\) To add an action to your lifecycle hook, [follow these steps](configuring-lifecycle-hook-notifications.md) to configure an AWS Lambda function or another target\. Otherwise, for an EC2 instance to run an action automatically, you must configure it to run scripts\. We recommend that you script your shutdown sequence to be completed in under one to two minutes to ensure that there is enough time to complete tasks before instance termination\. 

For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\. 