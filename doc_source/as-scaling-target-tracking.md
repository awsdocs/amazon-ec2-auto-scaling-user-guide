# Target Tracking Scaling Policies<a name="as-scaling-target-tracking"></a>

Target tracking scaling policies simplify how you configure dynamic scaling\. You select a predefined metric or configure a customized metric, and set a target value\. Amazon EC2 Auto Scaling creates and manages the CloudWatch alarms that trigger the scaling policy and calculates the scaling adjustment based on the metric and the target value\. The scaling policy adds or removes capacity as required to keep the metric at, or close to, the specified target value\. In addition to keeping the metric close to the target value, a target tracking scaling policy also adjusts to the fluctuations in the metric due to a fluctuating load pattern and minimizes rapid fluctuations in the capacity of the Auto Scaling group\.

For example, you could use target tracking scaling to:

+ Configure a target tracking scaling policy to keep the average aggregate CPU utilization of your Auto Scaling group at 50 percent\.

+ Configure a target tracking scaling policy to keep the request count per target of your Elastic Load Balancing target group at 1000 for your Auto Scaling group\.

When specifying a customized metric, be aware that not all metrics work for target tracking\. The metric must be a valid utilization metric and describe how busy an instance is\. The metric value must increase or decrease proportionally to the number of instances in the Auto Scaling group so that the metric data can be used to proportionally scale out or in the number of instances\. For example, the CPU utilization of an Auto Scaling group \(that is, the Amazon EC2 metric `CPUUtilization` with the dimension `AutoScalingGroupName`\) works, if the load on the Auto Scaling group is distributed across the instances\. The following metrics do not work:

+ The number of requests received by the load balancer fronting the Auto Scaling group \(that is, the Elastic Load Balancing metric `RequestCount`\), because the number of requests received by the load balancer doesn’t change based on the utilization of the Auto Scaling group\.

+ Load balancer request latency \(that is, the Elastic Load Balancing metric `Latency`\), because request latency can increase based on increasing utilization, but doesn’t necessarily change proportionally\.

+ The SQS metric `ApproximateNumberOfMessagesVisible`, because the number of messages in a queue may not change proportionally to the size of the Auto Scaling group that processes messages from the queue\. However, if the messages are distributed across the instances, a custom metric that measures the number of messages in the queue per instance of the Auto Scaling group would work\.

You can have multiple target tracking scaling policies for an Auto Scaling group, provided that each of them is using a different metric\. The Auto Scaling group scales based on the policy that provides the largest capacity in the group for both scale in and scale out\. This allows you greater flexibility to cover multiple scenarios and ensure that there is always enough capacity to process your application workloads\.

You can also optionally disable the scale\-in portion of a target tracking scaling policy\. This feature provides you with the flexibility to scale in your Auto Scaling group using a different method, for example a different scaling policy type, while using a target tracking scaling policy to scale out your Auto Scaling group\.

## Considerations<a name="target-tracking-considerations"></a>

Keep the following considerations in mind:

+ A target tracking scaling policy assumes that it should scale out your Auto Scaling group when the specified metric is above the target value\. You cannot use a target tracking scaling policy to scale out your Auto Scaling group when the specified metric is below the target value\.

+ A target tracking scaling policy does not scale your Auto Scaling group when its current capacity is 0, because target tracking scales proportionally to the current capacity\.

+ A target tracking scaling policy does not scale your Auto Scaling group when the specified metric has insufficient data\. It does not scale in your Auto Scaling group because it does not interpret insufficient data as low utilization\. To scale in your Auto Scaling group when the specified metric has insufficient data, create a simple or step scaling policy and have an alarm invoke the scaling policy when it changes to the `INSUFFICIENT_DATA` state\. For example, the metric `RequestCountPerTarget`, which is one of the predefined metrics, has no data points when no requests are routed to the target group\. To scale in your Auto Scaling group when no requests are routed to the target group, create a simple or step scaling policy, create an alarm on the metric, and have it invoke the scaling policy when it changes to the `INSUFFICIENT_DATA` state\.

+ You may see gaps between the target value and the actual metric data points\. This is because we act conservatively by rounding up or down when determining how many instances to add or remove\. This prevents us from adding an insufficient number of instances or removing too many instances\. However, for smaller Auto Scaling groups with fewer instances, the utilization of the group may seem far from the target value\. For example, if you set a target value of 50 percent for CPU utilization and your Auto Scaling group then exceeds the target, we might determine that adding 1\.5 instances would decrease the CPU utilization to close to 50 percent\. Because it is not possible to add 1\.5 instances, we round up and add two instances\. This might decrease the CPU utilization to a value below 50 percent, but it ensures that your application has enough resources to support it\. Similarly, if we determine that removing 1\.5 instances increases your CPU utilization to above 50 percent, we remove just one instance\. For larger Auto Scaling groups with more instances, the utilization is spread over a larger number of instances\. Adding or removing instances causes less of a gap between the target value and the actual metric data points\.

+ We recommend that you scale on metrics with a 1\-minute frequency because that ensures a faster response to utilization changes\. Scaling on metrics with a 5\-minute frequency can result in slower response time and scaling on stale metric data\. By default, Amazon EC2 instances are enabled for basic monitoring, which means metric data for instances is available at 5\-minute intervals\. You can enable detailed monitoring to get metric data for instances at 1\-minute frequency\. For more information, see [Configure Monitoring for Auto Scaling Instances](http://docs.aws.amazon.com/autoscaling/ec2/userguide/as-instance-monitoring.html)\.

+ To ensure application availability, the Auto Scaling group scales out proportionally to the metric as fast as it can, but scales in more gradually\.

+ Do not edit or delete the CloudWatch alarms that Amazon EC2 Auto Scaling manages for a target tracking scaling policy\. The alarms are deleted automatically when you delete the scaling policy\.

## Create an Auto Scaling Group with Target Tracking Scaling Policies<a name="policy_creating"></a>

Use the console to create an Auto Scaling group with a target tracking scaling policy\.

**To create an Auto Scaling group with scaling based on metrics**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, do one of the following:

   + Select **Create an Auto Scaling group from an existing launch configuration**, select an existing launch configuration, and then choose **Next Step**\.

   + If you don't have a launch configuration that you'd like to use, choose **Create a new launch configuration** and follow the directions\. For more information, see [Creating a Launch Configuration](create-launch-config.md)\.

1. On the **Configure Auto Scaling group details** page, do the following:

   1. For **Group name**, type a name for your Auto Scaling group\.

   1. For **Group size**, type the desired capacity for your Auto Scaling group\.

   1. If the launch configuration specifies instances that require a VPC, such as T2 instances, you must select a VPC from **Network**\. Otherwise, if your AWS account supports EC2\-Classic and the instances don't require a VPC, you can select either `Launch info EC2-Classic` or a VPC\.

   1. If you selected a VPC in the previous step, select one or more subnets from **Subnet**\. If you selected EC2\-Classic in the previous step, select one or more Availability Zones from **Availability Zone\(s\)**\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, do the following:

   1. Select **Use scaling policies to adjust the capacity of this group**\.

   1. Specify the minimum and maximum size for your Auto Scaling group using the row that begins with **Scale between**\. For example, if your group is already at its maximum size, specify a new maximum in order to scale out\.

   1. For **Scale Group Size**, specify a scaling policy\. You can optionally specify a name for  the policy, then choose a value for **Metric  type**\.

   1. Specify a **Target value** for the  metric\.

   1. Specify an instance warm\-up value for **Instances  need**, which allows you to control the time until a newly launched instance can contribute to the CloudWatch metrics\.

   1. Check the **Disable scale\-in** option to create only a scale\-out policy\. This allows you to create a separate scale\-in policy of a different type if wanted\.

   1. Choose **Review**\.

   1. On the **Review** page, choose  **Create Auto Scaling group**\.

## Instance Warmup<a name="as-target-tracking-scaling-warmup"></a>

With target tracking scaling policies, you can specify the number of seconds that it takes for a newly launched instance to warm up\. Until its specified warm\-up time has expired, an instance is not counted toward the aggregated metrics of the Auto Scaling group\.

While scaling out, we do not consider instances that are warming up as part of the current capacity of the group\. This ensures that we don't add more instances than you need\.

While scaling in, we consider instances that are terminating as part of the current capacity of the group\. Therefore, we don't remove more instances from the Auto Scaling group than necessary\.

A scale\-in activity can't start while a scale\-out activity is in progress\.

## Configure Scaling Policies Using the AWS CLI<a name="target-tracking-policy-creating-aws-cli"></a>

Use the AWS CLI as follows to configure target tracking scaling policies for your Auto Scaling group\.


+ [Step 1: Create an Auto Scaling Group](#policy-creating-tt-aws-cli)
+ [Step 2: Create Scaling Policies](#policy-creating-ttt-aws-cli)

### Step 1: Create an Auto Scaling Group<a name="policy-creating-tt-aws-cli"></a>

Use the following [create\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group named `my-asg` using the launch configuration `my-lc`\. If you don't have a launch configuration that you'd like to use, you can create one\. For more information, see [create\-launch\-configuration](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html)\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg --launch-configuration-name my-lc --max-size 5 --min-size 1 --availability-zones "us-west-2c"
```

### Step 2: Create Scaling Policies<a name="policy-creating-ttt-aws-cli"></a>

You can create scaling policies that tell the Auto Scaling group what to do when the specified conditions change\.

**Example: target tracking configuration file**  
The following is an example of a target tracking configuration file, which you should save as `config.json`:

```
{
  "TargetValue": 40.0,
  "PredefinedMetricSpecification": 
    {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    }
}
```

**Example: my\-scaleout\-policy**  
Use the following [put\-scaling\-policy](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command, along with the `config.json` file you created previously, to create a scaling policy named `cpu40` that keeps the average CPU utilization of the Auto Scaling group at 40 percent:

```
aws autoscaling put-scaling-policy --policy-name cpu40 --auto-scaling-group-name my-asg --policy-type TargetTrackingScaling --target-tracking-configuration file://config.json
```