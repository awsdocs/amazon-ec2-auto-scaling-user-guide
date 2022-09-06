# Target tracking scaling policies for Amazon EC2 Auto Scaling<a name="as-scaling-target-tracking"></a>

To create a target tracking scaling policy, you specify an Amazon CloudWatch metric and a target value that represents the ideal average utilization or throughput level for your application\. Amazon EC2 Auto Scaling can then scale out your group \(add more instances\) to handle peak traffic, and scale in your group \(run fewer instances\) to reduce costs during periods of low utilization or throughput\.

For example, let's say that you currently have an application that runs on two instances, and you want the CPU utilization of the Auto Scaling group to stay at around 50 percent when the load on the application changes\. This gives you extra capacity to handle traffic spikes without maintaining an excessive number of idle resources\. 

You can meet this need by creating a target tracking scaling policy that targets an average CPU utilization of 50 percent\. Then, your Auto Scaling group scales the number of instances to keep the actual metric value at or near 50 percent\. 

**Topics**
+ [Multiple target tracking scaling policies](#target-tracking-multiple-policies)
+ [Considerations](#target-tracking-considerations)
+ [Choose metrics](#target-tracking-choose-metrics)
+ [Define target value](#target-tracking-define-target-value)
+ [Define instance warm\-up time](#as-target-tracking-scaling-warmup)
+ [Create a target tracking scaling policy \(console\)](#policy_creating)
+ [Create a target tracking scaling policy \(AWS CLI\)](#target-tracking-policy-creating-aws-cli)

## Multiple target tracking scaling policies<a name="target-tracking-multiple-policies"></a>

To help optimize scaling performance, you can use multiple target tracking scaling policies together, provided that each of them uses a different metric\. For example, utilization and throughput can influence each other\. Whenever one of these metrics changes, it usually implies that other metrics will also be impacted\. The use of multiple metrics therefore provides additional information about the load that your Auto Scaling group is under and improves decision making when determining how much capacity to add to your group\. 

The intention of Amazon EC2 Auto Scaling is to always prioritize availability, so its behavior differs depending on whether the target tracking policies are ready for scale out or scale in\. It will scale out the Auto Scaling group if any of the target tracking policies are ready for scale out, but will scale in only if all of the target tracking policies \(with the scale\-in portion enabled\) are ready to scale in\.

## Considerations<a name="target-tracking-considerations"></a>

The following considerations apply when working with target tracking scaling policies:
+ Do not create, edit, or delete the CloudWatch alarms that are used with a target tracking scaling policy\. Amazon EC2 Auto Scaling creates and manages the CloudWatch alarms that are associated with your target tracking scaling policies and deletes them when no longer needed\. 
+ A target tracking scaling policy prioritizes availability during periods of fluctuating traffic levels by scaling in more gradually when traffic is decreasing\. If you want your Auto Scaling group to scale in immediately when a workload finishes, you can disable the scale\-in portion of the policy\. This provides you with the flexibility to use the scale\-in method that best meets your needs when utilization is low\. To ensure that scale in happens as quickly as possible, we recommend not using a simple scaling policy to prevent a cooldown period from being added\.
+ If the metric is missing data points, this causes the CloudWatch alarm state to change to `INSUFFICIENT_DATA`\. When this happens, Amazon EC2 Auto Scaling cannot scale your group until new data points are found\.
+ You might see gaps between the target value and the actual metric data points\. This is because we act conservatively by rounding up or down when determining how many instances to add or remove\. This prevents us from adding an insufficient number of instances or removing too many instances\. However, for smaller Auto Scaling groups with fewer instances, the utilization of the group might seem far from the target value\. For example, let's say that you set a target value of 50 percent for CPU utilization and your Auto Scaling group then exceeds the target\. We might determine that adding 1\.5 instances will decrease the CPU utilization to close to 50 percent\. Because it is not possible to add 1\.5 instances, we round up and add two instances\. This might decrease the CPU utilization to a value below 50 percent, but it ensures that your application has enough resources to support it\. Similarly, if we determine that removing 1\.5 instances increases your CPU utilization to above 50 percent, we remove just one instance\. 
+ For larger Auto Scaling groups with more instances, the utilization is spread over a larger number of instances, in which case adding or removing instances causes less of a gap between the target value and the actual metric data points\.
+ A target tracking scaling policy assumes that it should scale out your Auto Scaling group when the specified metric is above the target value\. You can't use a target tracking scaling policy to scale out your Auto Scaling group when the specified metric is below the target value\.

## Choose metrics<a name="target-tracking-choose-metrics"></a>

In a target tracking scaling policy, you can use predefined or customized metrics\. 

The following predefined metrics are available: 
+ `ASGAverageCPUUtilization`—Average CPU utilization of the Auto Scaling group\.
+ `ASGAverageNetworkIn`—Average number of bytes received by a single instance on all network interfaces\.
+ `ASGAverageNetworkOut`—Average number of bytes sent out from a single instance on all network interfaces\.
+ `ALBRequestCountPerTarget`—Average Application Load Balancer request count per target\.

**Important**  
Other valuable information about the metrics for CPU utilization, network I/O, and Application Load Balancer request count per target can be found in the [List the available CloudWatch metrics for your instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html) topic in the *Amazon EC2 User Guide for Linux Instances* and the [CloudWatch metrics for your Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html) topic in the *User Guide for Application Load Balancers*, respectively\.

You can choose other available CloudWatch metrics or your own metrics in CloudWatch by specifying a customized metric\. You must use the AWS CLI or an SDK to create a target tracking policy with a customized metric\.

Keep the following in mind when choosing a metric:
+ Not all metrics work for target tracking\. This can be important when you are specifying a customized metric\. The metric must be a valid utilization metric and describe how busy an instance is\. The metric value must increase or decrease proportionally to the number of instances in the Auto Scaling group\. That's so the metric data can be used to proportionally scale out or in the number of instances\. For example, the CPU utilization of an Auto Scaling group works \(that is, the Amazon EC2 metric `CPUUtilization` with the metric dimension `AutoScalingGroupName`\), if the load on the Auto Scaling group is distributed across the instances\. 
+ The following metrics do not work for target tracking:
  + The number of requests received by the load balancer fronting the Auto Scaling group \(that is, the Elastic Load Balancing metric `RequestCount`\)\. The number of requests received by the load balancer doesn't change based on the utilization of the Auto Scaling group\.
  + Load balancer request latency \(that is, the Elastic Load Balancing metric `Latency`\)\. Request latency can increase based on increasing utilization, but doesn't necessarily change proportionally\.
  + The CloudWatch Amazon SQS queue metric `ApproximateNumberOfMessagesVisible`\. The number of messages in a queue might not change proportionally to the size of the Auto Scaling group that processes messages from the queue\. However, a customized metric that measures the number of messages in the queue per EC2 instance in the Auto Scaling group can work\. For more information, see [Scaling based on Amazon SQS](as-using-sqs-queue.md)\.
+ To use the `ALBRequestCountPerTarget` metric, you must specify the `ResourceLabel` parameter to identify the load balancer target group that is associated with the metric\. 
+ When a metric emits real 0 values to CloudWatch \(for example, `ALBRequestCountPerTarget`\), an Auto Scaling group can scale in to 0 when there is no traffic to your application\. To have your Auto Scaling group scale in to 0 when no requests are routed it, the group's minimum capacity must be set to 0\.

When you use EC2 instance metrics in your policy, we recommend that you configure these metrics with a 1\-minute granularity to ensure a faster response to changes in the metric value\. Scaling on instance metrics with a 5\-minute granularity can result in slower response times and scaling on stale metric data\.

To get this level of data for Amazon EC2 metrics, you must specifically enable detailed monitoring\. By default, Amazon EC2 instances are enabled for basic monitoring, which means metric data for instances is available at 5\-minute granularity\. For more information, see [Configure monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\.

## Define target value<a name="target-tracking-define-target-value"></a>

When you create a target tracking scaling policy, you must specify a target value\. The target value represents the optimal average utilization or throughput for the Auto Scaling group\. To use resources cost efficiently, set the target value as high as possible with a reasonable buffer for unexpected traffic increases\. When your application is scaled out to multiple instances, the actual metric value for a normal traffic flow should be at or just below the target value\.

When a scaling policy is based on throughput, such as the request count per target for an Application Load Balancer, network I/O, or other count metrics, the target value represents the optimal average throughput from a single instance, for a one\-minute period\.

## Define instance warm\-up time<a name="as-target-tracking-scaling-warmup"></a>

**Important**  
We recommend using the default instance warm\-up setting, which unifies all the warm\-up and cooldown settings for your Auto Scaling group\. For more information, see [Available warm\-up and cooldown settings](consolidated-view-of-warm-up-and-cooldown-settings.md)\.

You can optionally specify the number of seconds that it takes for a newly launched instance to warm up\. Until its specified warm\-up time has expired, an instance is not counted toward the aggregated EC2 instance metrics of the Auto Scaling group\.

While instances are in the warm\-up period, your scaling policies only scale out if the metric value from instances that are not warming up is greater than the policy's target utilization\.

If the group scales out again, the instances that are still warming up are counted as part of the desired capacity for the next scale\-out activity\. The intention is to continuously \(but not excessively\) scale out\.

While the scale\-out activity is in progress, all scale\-in activities initiated by scaling policies are blocked until the instances finish warming up\.

## Create a target tracking scaling policy \(console\)<a name="policy_creating"></a>

You can choose to configure a target tracking scaling policy on an Auto Scaling group as you create it or after the Auto Scaling group is created\.

**To create an Auto Scaling group with a target tracking scaling policy**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Choose **Create Auto Scaling group**\.

1.  In Steps 1, 2, and 3, choose the options as desired and proceed to **Step 4: Configure group size and scaling policies**\.

1. Under **Group size**, specify the range that you want to scale between by updating the minimum capacity and maximum capacity\. These two settings allow your Auto Scaling group to scale dynamically\. Amazon EC2 Auto Scaling scales your group in the range of values specified by the minimum capacity and maximum capacity\.

1. Under **Scaling policies**, choose **Target tracking scaling policy**\.

1. To define a policy, do the following:

   1. Specify a name for the policy\.

   1. For **Metric type**, choose a metric\. 

      If you chose **Application Load Balancer request count per target**, choose a target group in **Target group**\.

   1. Specify a **Target value** for the metric\.

   1. \(Optional\) For **Instances need**, update the instance warm\-up value as needed\.

   1. \(Optional\) Select **Disable scale in to create only a scale\-out policy**\. This allows you to create a separate scale\-in policy of a different type if wanted\.

1. Proceed to create the Auto Scaling group\. Your scaling policy will be created after the Auto Scaling group has been created\. 

**To create a target tracking scaling policy for an existing Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. Verify that the minimum capacity and maximum capacity are appropriately set\. For example, if your group is already at its maximum size, specify a new maximum in order to scale out\. Amazon EC2 Auto Scaling does not scale your group below the minimum capacity or above the maximum capacity\. To update your group, on the **Details** tab, change the current settings for minimum and maximum capacity\. 

1. On the **Automatic scaling** tab, in **Dynamic scaling policies**, choose **Create dynamic scaling policy**\.

1. To define a policy, do the following:

   1. For **Policy type**, leave the default of **Target tracking scaling**\. 

   1. Specify a name for the policy\.

   1. For **Metric type**, choose a metric\. You can choose only one metric type\. To use more than one metric, create multiple policies\.

      If you chose **Application Load Balancer request count per target**, choose a target group in **Target group**\.

   1. Specify a **Target value** for the metric\.

   1. \(Optional\) For **Instances need**, update the instance warm\-up value as needed\.

   1. \(Optional\) Select **Disable scale in to create only a scale\-out policy**\. This allows you to create a separate scale\-in policy of a different type if wanted\.

1. Choose **Create**\.

## Create a target tracking scaling policy \(AWS CLI\)<a name="target-tracking-policy-creating-aws-cli"></a>

Use the AWS CLI as follows to configure target tracking scaling policies for your Auto Scaling group\.

**Topics**
+ [Step 1: Create an Auto Scaling group](#policy-creating-tt-aws-cli)
+ [Step 2: Create a target tracking scaling policy](#policy-creating-ttt-aws-cli)

### Step 1: Create an Auto Scaling group<a name="policy-creating-tt-aws-cli"></a>

Use the [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group named `my-asg` using the launch template `my-template`\. If you don't have a launch template, see [AWS CLI examples for working with launch templates](examples-launch-templates-aws-cli.md)\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template,Version='2' \
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
Use the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command, along with the `config.json` file that you created previously, to create a scaling policy named `cpu40-target-tracking-scaling-policy` that keeps the average CPU utilization of the Auto Scaling group at 40 percent\.

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