# Target tracking scaling policies for Amazon EC2 Auto Scaling<a name="as-scaling-target-tracking"></a>

With target tracking scaling policies, you select a scaling metric and set a target value\. Amazon EC2 Auto Scaling creates and manages the CloudWatch alarms that trigger the scaling policy and calculates the scaling adjustment based on the metric and the target value\. The scaling policy adds or removes capacity as required to keep the metric at, or close to, the specified target value\. In addition to keeping the metric close to the target value, a target tracking scaling policy also adjusts to changes in the metric due to a changing load pattern\. 

For example, you can use target tracking scaling to:
+ Configure a target tracking scaling policy to keep the average aggregate CPU utilization of your Auto Scaling group at 40 percent\.
+ Configure a target tracking scaling policy to keep the request count per target of your Application Load Balancer target group at 1000 for your Auto Scaling group\.

Depending on your application needs, you might find that one of these popular scaling metrics works best for you when using target tracking, or you might find that a combination of these metrics or a different metric meets your needs better\.

**Contents**
+ [Considerations](#target-tracking-considerations)
  + [Choosing metrics](#available-metrics)
  + [Monitoring Amazon EC2 metrics](#target-tracking-monitoring)
  + [Instance warm\-up](#as-target-tracking-scaling-warmup)
+ [Create a target tracking scaling policy \(console\)](#policy_creating)
+ [Create a target tracking scaling policy \(AWS CLI\)](#target-tracking-policy-creating-aws-cli)
  + [Step 1: Create an Auto Scaling group](#policy-creating-tt-aws-cli)
  + [Step 2: Create a target tracking scaling policy](#policy-creating-ttt-aws-cli)

## Considerations<a name="target-tracking-considerations"></a>

Before you create a target tracking scaling policy for your Auto Scaling group, you should understand the following characteristics and behaviors of target tracking scaling policies:
+ A target tracking scaling policy assumes that it should scale out your Auto Scaling group when the specified metric is above the target value\. You cannot use a target tracking scaling policy to scale out your Auto Scaling group when the specified metric is below the target value\.
+ You might see gaps between the target value and the actual metric data points\. This is because we act conservatively by rounding up or down when determining how many instances to add or remove\. This prevents us from adding an insufficient number of instances or removing too many instances\. However, for smaller Auto Scaling groups with fewer instances, the utilization of the group might seem far from the target value\. For example, let's say that you set a target value of 50 percent for CPU utilization and your Auto Scaling group then exceeds the target\. We might determine that adding 1\.5 instances will decrease the CPU utilization to close to 50 percent\. Because it is not possible to add 1\.5 instances, we round up and add two instances\. This might decrease the CPU utilization to a value below 50 percent, but it ensures that your application has enough resources to support it\. Similarly, if we determine that removing 1\.5 instances increases your CPU utilization to above 50 percent, we remove just one instance\. 
+ For larger Auto Scaling groups with more instances, the utilization is spread over a larger number of instances, in which case adding or removing instances causes less of a gap between the target value and the actual metric data points\.
+ To ensure application availability, the Auto Scaling group scales out proportionally to the metric as fast as it can, but scales in more gradually\.
+ You can have multiple target tracking scaling policies for an Auto Scaling group, provided that each of them uses a different metric\. The intention of Amazon EC2 Auto Scaling is to always prioritize availability, so its behavior differs depending on whether the target tracking policies are ready for scale out or scale in\. It will scale out the Auto Scaling group if any of the target tracking policies are ready for scale out, but will scale in only if all of the target tracking policies \(with the scale\-in portion enabled\) are ready to scale in\. For more information, see [Multiple scaling policies](as-scale-based-on-demand.md#multiple-scaling-policy-resolution)\. 
+ You can disable the scale\-in portion of a target tracking scaling policy\. This feature provides you with the flexibility to scale in your Auto Scaling group using a different method\. For example, you can use a different scaling policy type for scale in while using a target tracking scaling policy for scale out\. 
+ Do not edit or delete the CloudWatch alarms that are configured for the target tracking scaling policy\. CloudWatch alarms that are associated with your target tracking scaling policies are managed by AWS and deleted automatically when no longer needed\. 

### Choosing metrics<a name="available-metrics"></a>

In a target tracking scaling policy, you can use predefined or customized metrics\. 

The following predefined metrics are available: 
+ `ASGAverageCPUUtilization`—Average CPU utilization of the Auto Scaling group\. 
+ `ASGAverageNetworkIn`—Average number of bytes received on all network interfaces by the Auto Scaling group\. 
+ `ASGAverageNetworkOut`—Average number of bytes sent out on all network interfaces by the Auto Scaling group\. 
+ `ALBRequestCountPerTarget`—Number of requests completed per target in an Application Load Balancer target group\. 

You can choose other available Amazon CloudWatch metrics or your own metrics in CloudWatch by specifying a customized metric\. You must use the AWS CLI or an AWS SDK to create a target tracking policy with a customized metric\.

Keep the following in mind when choosing a metric:
+ Not all metrics work for target tracking\. This can be important when you are specifying a customized metric\. The metric must be a valid utilization metric and describe how busy an instance is\. The metric value must increase or decrease proportionally to the number of instances in the Auto Scaling group\. That's so the metric data can be used to proportionally scale out or in the number of instances\. For example, the CPU utilization of an Auto Scaling group works \(that is, the Amazon EC2 metric `CPUUtilization` with the metric dimension `AutoScalingGroupName`\), if the load on the Auto Scaling group is distributed across the instances\. 
+ The following metrics do not work for target tracking:
  + The number of requests received by the load balancer fronting the Auto Scaling group \(that is, the Elastic Load Balancing metric `RequestCount`\)\. The number of requests received by the load balancer doesn't change based on the utilization of the Auto Scaling group\.
  + Load balancer request latency \(that is, the Elastic Load Balancing metric `Latency`\)\. Request latency can increase based on increasing utilization, but doesn't necessarily change proportionally\.
  + The CloudWatch Amazon SQS queue metric `ApproximateNumberOfMessagesVisible`\. The number of messages in a queue might not change proportionally to the size of the Auto Scaling group that processes messages from the queue\. However, a customized metric that measures the number of messages in the queue per EC2 instance in the Auto Scaling group can work\. For more information, see [Scaling based on Amazon SQS](as-using-sqs-queue.md)\.
+ A target tracking scaling policy does not scale in your Auto Scaling group when the specified metric has insufficient data, unless you use the `ALBRequestCountPerTarget` metric\. This works because the `ALBRequestCountPerTarget` metric emits zeros for periods with no associated data, and the target tracking policy requires metric data to interpret a low utilization trend\. To have your Auto Scaling group scale in to 0 instances when no requests are routed to the target group, the group's minimum capacity must be set to 0\. 
+ To use the `ALBRequestCountPerTarget` metric, you must specify the `ResourceLabel` parameter to identify the target group that is associated with the metric\. 

### Monitoring Amazon EC2 metrics<a name="target-tracking-monitoring"></a>

To ensure a faster response to changes in the metric value, we recommend that you scale on metrics with a 1\-minute frequency\. Scaling on metrics with a 5\-minute frequency can result in slower response times and scaling on stale metric data\. 

To get this level of data for Amazon EC2 metrics, you must specifically enable detailed monitoring\. By default, Amazon EC2 instances are enabled for basic monitoring, which means metric data for instances is available at 5\-minute frequency\. For more information, see [Configuring monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\.

### Instance warm\-up<a name="as-target-tracking-scaling-warmup"></a>

When you create a target tracking scaling policy, you can specify the number of seconds that it takes for a newly launched instance to warm up\. Until its specified warm\-up time has expired, an instance is not counted toward the aggregated metrics of the Auto Scaling group\. 

While scaling out, we do not consider instances that are warming up as part of the current capacity of the group\. This ensures that we don't add more instances than you need\.

While scaling in, we consider instances that are terminating as part of the current capacity of the group\. Therefore, we don't remove more instances from the Auto Scaling group than necessary\.

A scale\-in activity can't start while a scale\-out activity is in progress\.

## Create a target tracking scaling policy \(console\)<a name="policy_creating"></a>

**To create a target tracking scaling policy**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. Verify that the minimum and maximum size limits are appropriately set\. For example, if your group is already at its maximum size, specify a new maximum in order to scale out\. Amazon EC2 Auto Scaling does not scale your group below the minimum capacity or above the maximum capacity\. To update your group, on the **Details** tab, change the current settings for minimum and maximum capacity\. 

1. On the **Automatic scaling** tab, in **Scaling policies**, choose **Add policy**\. \(Old console: The **Scaling Policies** tab is where you can add the policy\.\) 

1. <a name="policy-creating-scalingpolicies-console"></a>To define a policy, do the following:

   1. For **Policy type**, leave the default of **Target tracking scaling**\. 

   1. Specify a name for the policy\.

   1. For **Metric type**, choose a metric\. You can choose only one metric type\. To use more than one metric, create multiple policies\.

   1. For **Target group**, choose the target group that you specified in the Auto Scaling group's load balancer settings\. You need to complete this step only if you chose the metric type that is based on the request count per target of your Application Load Balancer\.

   1. Specify a **Target value** for the metric\.

   1. \(Optional\) Specify an instance warm\-up value for **Instances need**\. This allows you to control the time until a newly launched instance can contribute to the CloudWatch metrics\.

   1. \(Optional\) Select **Disable scale in to create only a scale\-out policy**\. This allows you to create a separate scale\-in policy of a different type if wanted\.

1. Choose **Create**\.

## Create a target tracking scaling policy \(AWS CLI\)<a name="target-tracking-policy-creating-aws-cli"></a>

Use the AWS CLI as follows to configure target tracking scaling policies for your Auto Scaling group\.

**Topics**
+ [Step 1: Create an Auto Scaling group](#policy-creating-tt-aws-cli)
+ [Step 2: Create a target tracking scaling policy](#policy-creating-ttt-aws-cli)

### Step 1: Create an Auto Scaling group<a name="policy-creating-tt-aws-cli"></a>

Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group named `my-asg` using the launch configuration `my-launch-config`\. If you don't have a launch configuration that you'd like to use, you can create one by calling [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html)\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --max-size 5 --min-size 1
```

### Step 2: Create a target tracking scaling policy<a name="policy-creating-ttt-aws-cli"></a>

After you have created the Auto Scaling group, you can create a target tracking scaling policy that instructs Amazon EC2 Auto Scaling to increase and decrease the number of running EC2 instances in the group dynamically when the load on the application changes\. 

**Example: target tracking configuration file**  
The following is an example target tracking configuration that keeps the average CPU utilization at 40 percent\. Save this configuration in a file named `config.json`\.

```
{
  "TargetValue": 40.0,
  "PredefinedMetricSpecification": 
    {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    }
}
```

For more information, see [PredefinedMetricSpecification](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PredefinedMetricSpecification.html) in the *Amazon EC2 Auto Scaling API Reference*\.

Alternatively, you can customize the metric used for scaling by creating a customized metric specification and adding values for each parameter from CloudWatch\. The following is an example target tracking configuration that keeps the average utilization of the specified metric at 40 percent\.

```
{
   "TargetValue":40.0,
   "CustomizedMetricSpecification":{
      "MetricName":"MyUtilizationMetric",
      "Namespace":"MyNamespace",
      "Dimensions":[
         {
            "Name":"MyOptionalMetricDimensionName",
            "Value":"MyOptionalMetricDimensionValue"
         }
      ],
      "Statistic":"Average",
      "Unit":"Percent"
   }
}
```

For more information, see [CustomizedMetricSpecification](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CustomizedMetricSpecification.html) in the *Amazon EC2 Auto Scaling API Reference*\.

**Example: cpu40\-target\-tracking\-scaling\-policy**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command, along with the `config.json` file that you created previously, to create a scaling policy named `cpu40-target-tracking-scaling-policy` that keeps the average CPU utilization of the Auto Scaling group at 40 percent\.

```
aws autoscaling put-scaling-policy --policy-name cpu40-target-tracking-scaling-policy \
  --auto-scaling-group-name my-asg --policy-type TargetTrackingScaling \
  --target-tracking-configuration file://config.json
```

If successful, this command returns the ARNs and names of the two CloudWatch alarms created on your behalf\.

```
{
    "PolicyARN": "arn:aws:autoscaling:region:account-id:scalingPolicy:228f02c2-c665-4bfd-aaac-8b04080bea3c:autoScalingGroupName/my-asg:policyName/cpu40-target-tracking-scaling-policy",
    "Alarms": [
        {
            "AlarmARN": "arn:aws:cloudwatch:region:account-id:alarm:TargetTracking-my-asg-AlarmHigh-fc0e4183-23ac-497e-9992-691c9980c38e",
            "AlarmName": "TargetTracking-my-asg-AlarmHigh-fc0e4183-23ac-497e-9992-691c9980c38e"
        },
        {
            "AlarmARN": "arn:aws:cloudwatch:region:account-id:alarm:TargetTracking-my-asg-AlarmLow-61a39305-ed0c-47af-bd9e-471a352ee1a2",
            "AlarmName": "TargetTracking-my-asg-AlarmLow-61a39305-ed0c-47af-bd9e-471a352ee1a2"
        }
    ]
}
```