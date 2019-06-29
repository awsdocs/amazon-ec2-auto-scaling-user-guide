# Simple and Step Scaling Policies for Amazon EC2 Auto Scaling<a name="as-scaling-simple-step"></a>

With simple and step scaling policies, you choose scaling metrics and threshold values for the CloudWatch alarms that trigger the scaling process as well as define how your Auto Scaling group should be scaled when a threshold is in breach for a specified number of evaluation periods\. 

We recommend that you use step scaling policies instead of simple scaling policies, even if you have a single scaling adjustment\. Amazon EC2 Auto Scaling originally supported only simple scaling policies\. If you created your scaling policy before target tracking and step policies were introduced, your policy is treated as a simple scaling policy\. For more information, see [Simple Scaling Policies and Cooldowns](#SimpleScaling)\.

If your scaling metric is a utilization metric that increases or decreases proportionally to the number of instances in the Auto Scaling group, we recommend that you use a target tracking scaling policy\. For more information, see [Target Tracking Scaling Policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)\. You still have the option to use target tracking scaling with step scaling for a more advanced scaling policy configuration\. For example, if you want, you can configure a more aggressive response when utilization reaches a certain level\.

## Step Scaling Policies<a name="StepScaling"></a>

Step scaling policies increase or decrease the current capacity of your Auto Scaling group based on a set of scaling adjustments, known as *step adjustments*\. The adjustments vary based on the size of the alarm breach\. 

After a scaling activity is started, the policy continues to respond to additional alarms, even while a scaling activity or health check replacement is in progress\. Therefore, all alarms that are breached are evaluated by Amazon EC2 Auto Scaling as it receives the alarm messages\. 

If you are creating a policy to scale out, you can specify the estimated warm\-up time that it takes for a newly launched instance to be ready to contribute to the aggregated metrics\. For more information, see [Instance Warmup](#as-step-scaling-warmup)\.

You can also use step scaling for resources beyond Amazon EC2\. For more information, see [Step Scaling Policies](https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling-step-scaling-policies.html) in the *Application Auto Scaling User Guide*\.

## Simple Scaling Policies and Cooldowns<a name="SimpleScaling"></a>

In most cases, step scaling policies are a better choice than simple scaling policies\. With simple scaling, after a scaling activity is started, the policy must wait for the scaling activity or health check replacement to complete and the cooldown period to expire before responding to additional alarms\. Cooldown periods help to prevent the initiation of additional scaling activities before the effects of previous activities are visible\. For more information, see [Scaling Cooldowns for Amazon EC2 Auto Scaling](Cooldown.md)\. 

Amazon EC2 Auto Scaling does not support cooldown periods for step scaling policies\. Therefore, you can't specify a cooldown period for these policies and the default cooldown period for the group doesn't apply\. 

## Scaling Adjustment Types<a name="as-scaling-adjustment"></a>

When a step scaling or simple scaling policy is executed, it changes the current capacity of your Auto Scaling group using the scaling adjustment specified in the policy\. A scaling adjustment can't change the capacity of the group above the maximum group size or below the minimum group size\.

Amazon EC2 Auto Scaling supports the following adjustment types for step scaling and simple scaling:
+ `ChangeInCapacity`—Increase or decrease the current capacity of the group by the specified number of instances\. A positive value increases the capacity and a negative adjustment value decreases the capacity\.

  Example: If the current capacity of the group is 3 instances and the adjustment is 5, then when this policy is performed, there are 5 instances added to the group for a total of 8 instances\.
+ `ExactCapacity`—Change the current capacity of the group to the specified number of instances\. Specify a positive value with this adjustment type\.

  Example: If the current capacity of the group is 3 instances and the adjustment is 5, then when this policy is performed, the capacity is set to 5 instances\.
+ `PercentChangeInCapacity`—Increment or decrement the current capacity of the group by the specified percentage\. A positive value increases the capacity and a negative value decreases the capacity\. If the resulting value is not an integer, it is rounded as follows:
  + Values greater than 1 are rounded down\. For example, `12.7` is rounded to `12`\.
  + Values between 0 and 1 are rounded to 1\. For example, `.67` is rounded to `1`\.
  + Values between 0 and \-1 are rounded to \-1\. For example, `-.58` is rounded to `-1`\.
  + Values less than \-1 are rounded up\. For example, `-6.67` is rounded to `-6`\.

  Example: If the current capacity is 10 instances and the adjustment is 10 percent, then when this policy is performed, 1 instance is added to the group for a total of 11 instances\.

With `PercentChangeInCapacity`, you can also specify the minimum number of instances to scale \(using the `MinAdjustmentMagnitude` parameter or **Add instances in increments of at least** in the console\)\. For example, suppose that you create a policy that adds 25 percent and you specify a minimum increment of 2 instances\. If you have an Auto Scaling group with 4 instances and the scaling policy is executed, 25 percent of 4 is 1 instance\. However, because you specified a minimum increment of 2, there are 2 instances added\.

## Step Adjustments<a name="as-scaling-steps"></a>

When you create a step scaling policy, you add one or more step adjustments that enable you to scale based on the size of the alarm breach\. Each step adjustment specifies the following:
+ A lower bound for the metric value
+ An upper bound for the metric value
+ The amount by which to scale, based on the scaling adjustment type

There are a few rules for the step adjustments for your policy:
+ The ranges of your step adjustments can't overlap or have a gap\.
+ Only one step adjustment can have a null lower bound \(negative infinity\)\. If one step adjustment has a negative lower bound, then there must be a step adjustment with a null lower bound\.
+ Only one step adjustment can have a null upper bound \(positive infinity\)\. If one step adjustment has a positive upper bound, then there must be a step adjustment with a null upper bound\.
+ The upper and lower bound can't be null in the same step adjustment\.
+ If the metric value is above the breach threshold, the lower bound is inclusive and the upper bound is exclusive\. If the metric value is below the breach threshold, the lower bound is exclusive and the upper bound is inclusive\.

CloudWatch aggregates metric data points based on the statistic for the metric associated with your CloudWatch alarm\. When the alarm is breached, the appropriate scaling policy is triggered\. Amazon EC2 Auto Scaling applies the aggregation type to the most recent metric data points from CloudWatch \(as opposed to the raw metric data\)\. It compares this aggregated metric value against the upper and lower bounds defined by the step adjustments to determine which step adjustment to perform\. 

If you are using the AWS Management Console, you specify the upper and lower bounds as absolute values\. If you are using the API or the CLI, you specify the upper and lower bounds relative to the breach threshold\. For example, suppose that you have an alarm with a breach threshold of 50 and a scaling adjustment type of `PercentChangeInCapacity`\. You also have scale\-out and scale\-in policies with the following step adjustments:


|  | 
| --- |
|  Scale out policy  | 
| Lower bound | Upper bound | Adjustment | Metric value | 
|  0  |  10  |  0  |  50 <= *value* < 60  | 
|  10  |  20  |  10  |  60 <= *value* < 70  | 
|  20  |  null  |  30  |  70 <= *value* < \+infinity  | 
|   Scale in policy   | 
| Lower bound | Upper bound | Adjustment | Metric value | 
|  \-10  |  0  |  0  |  40 < *value* <= 50  | 
|  \-20  |  \-10  |  \-10  |  30 < *value* <= 40  | 
|  null  |  \-20  |  \-30  |  \-infinity < *value* <= 30  | 

Your group has both a current capacity and a desired capacity of 10 instances\. The group maintains its current and desired capacity while the aggregated metric value is greater than 40 and less than 60\.

If the metric value gets to 60, the desired capacity of the group increases by 1 instance, to 11 instances, based on the second step adjustment of the scale\-out policy \(add 10 percent of 10 instances\)\. After the new instance is running and its specified warm\-up time has expired, the current capacity of the group increases to 11 instances\. If the metric value rises to 70 even after this increase in capacity, the desired capacity of the group increases by another 3 instances, to 14 instances, based on the third step adjustment of the scale\-out policy \(add 30 percent of 11 instances, 3\.3 instances, rounded down to 3 instances\)\.

If the metric value gets to 40, the desired capacity of the group decreases by 1 instance, to 13 instances, based on the second step adjustment of the scale\-in policy \(remove 10 percent of 14 instances, 1\.4 instances, rounded down to 1 instance\)\. If the metric value falls to 30 even after this decrease in capacity, the desired capacity of the group decreases by another 3 instances, to 10 instances, based on the third step adjustment of the scale\-in policy \(remove 30 percent of 13 instances, 3\.9 instances, rounded down to 3 instances\)\.

## Instance Warmup<a name="as-step-scaling-warmup"></a>

With step scaling policies, you can specify the number of seconds that it takes for a newly launched instance to warm up\. Until its specified warm\-up time has expired, an instance is not counted toward the aggregated metrics of the Auto Scaling group\. While scaling out, AWS also does not consider instances that are warming up as part of the current capacity of the group\. Therefore, multiple alarm breaches that fall in the range of the same step adjustment result in a single scaling activity\. This ensures that we don't add more instances than you need\. 

Using the example in the previous section, suppose that the metric gets to 60, and then it gets to 62 while the new instance is still warming up\. The current capacity is still 10 instances, so 1 instance is added \(10 percent of 10 instances\)\. However, the desired capacity of the group is already 11 instances, so AWS does not increase the desired capacity further\. If the metric gets to 70 while the new instance is still warming up, we should add 3 instances \(30 percent of 10 instances\)\. However, the desired capacity of the group is already 11, so we add only 2 instances, for a new desired capacity of 13 instances\.

While scaling in, AWS considers instances that are terminating as part of the current capacity of the group\. Therefore, we don't remove more instances from the Auto Scaling group than necessary\.

A scale\-in activity can't start while a scale\-out activity is in progress\.

## Create an Auto Scaling Group with Step Scaling Policies \(Console\)<a name="policy-creating-asg-console"></a>

You can create a scaling policy that uses CloudWatch alarms to determine when your Auto Scaling group should scale out or scale in\. Each CloudWatch alarm watches a single metric and sends messages to Amazon EC2 Auto Scaling when the metric breaches a threshold that you specify in your policy\. You can use alarms to monitor any of the metrics that are sent to CloudWatch from the services in AWS that you're using\. Or, you can create and monitor your own metrics\.

When you create a CloudWatch alarm, you can specify an Amazon SNS topic to send an email notification to when the alarm changes state\. For more information, see [Create Amazon CloudWatch Alarms](as-instance-monitoring.md#CloudWatchAlarm)\.

Use the console to create an Auto Scaling group with two scaling policies: a scale\-out policy that increases the capacity of the group by 30 percent, and a scale\-in policy that decreases the capacity of the group to two instances\.

**To create an Auto Scaling group with step scaling policies**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, do one of the following:
   + Select **Create an Auto Scaling group from an existing launch configuration**, select an existing launch configuration, and then choose **Next Step**\.
   + If you don't have a launch configuration that you'd like to use, choose **Create a new launch configuration** and follow the directions\. For more information, see [Creating a Launch Configuration](create-launch-config.md)\.

1. On the **Configure Auto Scaling group details** page, do the following:

   1. For **Group name**, type a name for your Auto Scaling group\.

   1. For **Group size**, type the desired capacity for your Auto Scaling group\.

   1. If the launch configuration specifies instances that require a VPC, such as T2 instances, you must select a VPC from **Network**\. Otherwise, if your AWS account supports EC2\-Classic and the instances don't require a VPC, you can select either `Launch into EC2-Classic` or a VPC\.

   1. If you selected a VPC in the previous step, select one or more subnets from **Subnet**\. If you selected EC2\-Classic in the previous step, select one or more Availability Zones from **Availability Zone\(s\)**\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, do the following:

   1. Select **Use scaling policies to adjust the capacity of this group**\.

   1. Specify the minimum and maximum size for your Auto Scaling group using the row that begins with **Scale between**\. For example, if your group is already at its maximum size, you need to specify a new maximum in order to scale out\.  
![\[Specify the minimum and maximum size for your group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-console-specify-min-max.png)

   1. Specify your scale\-out policy under **Increase Group Size**\. You can optionally specify a name for the policy, then choose **Add new alarm**\.

   1. On the **Create Alarm** page, choose **create topic**\. For **Send a notification to**, type a name for the SNS topic\. For **With these recipients**, type one or more email addresses to receive notification\. You can replace the default name for your alarm with a custom name\. Next, specify the metric and the criteria for the policy\. For example, you can leave the default settings for **Whenever** \(Average of CPU Utilization\)\. For **Is**, choose `>=` and type `80` percent\. For **For at least**, type `1` consecutive period of `5 Minutes`\. Choose **Create Alarm**\.  
![\[Create Alarm\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-console-create-scaleout-alarm.png)

   1. For **Take the action**, choose `Add`, type `30` in the next field, and then choose `percent of group`\. By default, the lower bound for this step adjustment is the alarm threshold and the upper bound is null \(positive infinity\)\.

      To add another step adjustment, choose **Add step**\. To set a minimum number of instances to scale, update the number field in **Add instances in increments of at least 1 instance\(s\)**\.

      \(Optional\) We recommend that you use the default to create scaling policies with steps\. To create simple scaling policies, choose **Create a simple scaling policy**\. For more information, see [Simple and Step Scaling Policies for Amazon EC2 Auto Scaling](#as-scaling-simple-step)\.  
![\[Create scale-out policy\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-console-create-scaleout-policy.png)

   1. Specify an instance warm\-up value for **Instances need**, which allows you to control the amount of time until a newly launched instance can contribute to the CloudWatch metrics\. 

   1. Specify your scale\-in policy under **Decrease Group Size**\. You can optionally specify a name for the policy, then choose **Add new alarm**\.

   1. On the **Create Alarm** page, you can select the same notification that you created for the scale\-out policy or create a new one for the scale\-in policy\. You can replace the default name for your alarm with a custom name\. Keep the default settings for **Whenever** \(Average of CPU Utilization\)\. For **Is**, choose `<=` and type `40` percent\. For **For at least**, type `1` consecutive period of `5 Minutes`\. Choose **Create Alarm**\.

   1. For **Take the action**, choose `Remove`, type `2` in the next field, and then choose `instances`\. By default, the upper bound for this step adjustment is the alarm threshold and the lower bound is null \(negative infinity\)\. To add another step adjustment, choose **Add step**\.

      \(Optional\) We recommend that you use the default to create scaling policies with steps\. To create simple scaling policies, choose **Create a simple scaling policy**\. For more information, see [Scaling Policy Types](as-scale-based-on-demand.md#as-scaling-types)\.  
![\[Create scale-out policy\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-console-create-scalein-policy.png)

   1. Choose **Review**\.

   1. On the **Review** page, choose **Create Auto Scaling group**\.

1. Use the following steps to verify the scaling policies for your Auto Scaling group\.

   1. The **Auto Scaling Group creation status** page confirms that your Auto Scaling group was successfully created\. Choose **View your Auto Scaling Groups**\.

   1. On the **Auto Scaling Groups** page, select the Auto Scaling group that you just created\.

   1. On the **Activity History** tab, the **Status** column shows whether your Auto Scaling group has successfully launched instances\.

   1. On the **Instances** tab, the **Lifecycle** column contains the state of your instances\. It takes a short time for an instance to launch\. After the instance starts, its lifecycle state changes to `InService`\.

      The **Health Status** column shows the result of the EC2 instance health check on your instance\.

   1. On the **Scaling Policies** tab, you can see the policies that you created for the Auto Scaling group\.

## Configure Step Scaling Policies \(AWS CLI\)<a name="policy-creating-aws-cli"></a>

Use the AWS CLI as follows to configure step scaling policies for your Auto Scaling group\.

**Topics**
+ [Step 1: Create an Auto Scaling Group](#policy-creating-asg-aws-cli)
+ [Step 2: Create Scaling Policies](#policy-creating-policy-aws-cli)
+ [Step 3: Create CloudWatch Alarms](#policy-creating-alarms-aws-cli)

### Step 1: Create an Auto Scaling Group<a name="policy-creating-asg-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group named `my-asg` using the launch configuration `my-launch-config`\. If you don't have a launch configuration that you'd like to use, you can create one\. For more information, see [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html)\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --max-size 5 --min-size 1
```

### Step 2: Create Scaling Policies<a name="policy-creating-policy-aws-cli"></a>

You can create step or simple scaling policies that tell the Auto Scaling group what to do when the load on the application changes\.

**Example: my\-step\-scaleout\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a step scaling policy named `my-step-scaleout-policy` with an adjustment type of `PercentChangeInCapacity` that increases the capacity of the group based on the following step adjustments \(assuming a CloudWatch alarm threshold of 60%\):
+ Increase the instance count by 10% when the value of the metric is greater than or equal to 70% but less than 80% 
+ Increase the instance count by 20% when the value of the metric is greater than or equal to 80% but less than 90%
+ Increase the instance count by 30% when the value of the metric is greater than or equal to 90%

```
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg  \
  --policy-name my-step-scaleout-policy \
  --policy-type StepScaling \
  --adjustment-type PercentChangeInCapacity \
  --metric-aggregation-type Average \
  --step-adjustments MetricIntervalLowerBound=10.0,MetricIntervalUpperBound=20.0,ScalingAdjustment=10 \
                     MetricIntervalLowerBound=20.0,MetricIntervalUpperBound=30.0,ScalingAdjustment=20 \
                     MetricIntervalLowerBound=30.0,ScalingAdjustment=30 \
  --min-adjustment-magnitude 1
```

The output includes the ARN for the policy\. Store this ARN in a safe place\. You need it to create CloudWatch alarms\.

```
{
    "PolicyARN": "arn:aws:autoscaling:region:123456789012:scalingPolicy:4ee9e543-86b5-4121-b53b-aa4c23b5bbcc:autoScalingGroupName/my-asg:policyName/my-step-scalein-policy
}
```

**Example: my\-step\-scalein\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a step scaling policy named `my-step-scalein-policy` with an adjustment type of `ChangeInCapacity` that decreases the capacity of the group by 2 instances:

```
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg  \
  --policy-name my-step-scalein-policy \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --step-adjustments MetricIntervalUpperBound=0.0,ScalingAdjustment=-2
```

The output includes the ARN that serves as a unique name for the policy\. Later, you can use either the ARN or a combination of the policy name and group name to specify the policy\. Store this ARN in a safe place\. You need it to create CloudWatch alarms\.

```
{
    "PolicyARN": "arn:aws:autoscaling:region:123456789012:scalingPolicy:ac542982-cbeb-4294-891c-a5a941dfa787:autoScalingGroupName/my-asg:policyName/my-step-scaleout-policy
}
```

#### Examples of Simple Scaling Policies<a name="policy-creating-aws-cli"></a>

Alternatively, you can create simple scaling policies by using the following CLI commands instead of the preceding CLI commands\. The output of these commands includes an ARN for each policy, which you need to create the CloudWatch alarms\. Keep in mind that a cooldown period will be in place due to the use of simple scaling policies\. 

**Example: my\-simple\-scaleout\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a simple scaling policy named `my-simple-scaleout-policy` with an adjustment type of `PercentChangeInCapacity` that increases the capacity of the group by 30 percent:

```
aws autoscaling put-scaling-policy --policy-name my-simple-scaleout-policy \
  --auto-scaling-group-name my-asg --scaling-adjustment 30 \
  --adjustment-type PercentChangeInCapacity
```

**Example: my\-simple\-scalein\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a simple scaling policy named `my-simple-scalein-policy` with an adjustment type of `ChangeInCapacity` that decreases the capacity of the group by two instances:

```
aws autoscaling put-scaling-policy --policy-name my-simple-scalein-policy \
  --auto-scaling-group-name my-asg --scaling-adjustment -2 \
  --adjustment-type ChangeInCapacity
```

### Step 3: Create CloudWatch Alarms<a name="policy-creating-alarms-aws-cli"></a>

In step 2, you created scaling policies that provided instructions to the Auto Scaling group about how to scale in and scale out when the conditions that you specify change\. In this step, you create alarms by identifying the metrics to watch, defining the conditions for scaling, and then associating the alarms with the scaling policies\. 

**Example: AddCapacity**  
Use the following CloudWatch [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to create an alarm that increases the size of the Auto Scaling group based on an average CPU threshold value of 60% for at least two consecutive evaluation periods of two minutes\. To use your own custom metric, specify its name in `--metric-name` and its namespace in `--namespace`\.

```
aws cloudwatch put-metric-alarm --alarm-name Step-Scaling-AlarmHigh-AddCapacity \
  --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average \
  --period 120 --evaluation-periods 2 --threshold 60 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --dimensions "Name=AutoScalingGroupName,Value=my-asg" \
  --alarm-actions PolicyARN
```

**Example: RemoveCapacity**  
Use the following CloudWatch [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to create an alarm that decreases the size of the Auto Scaling group based on average CPU threshold value of 40% for at least two consecutive evaluation periods of two minutes\. To use your own custom metric, specify its name in `--metric-name` and its namespace in `--namespace`\.

```
aws cloudwatch put-metric-alarm --alarm-name Step-Scaling-AlarmLow-RemoveCapacity \
  --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average \
  --period 120 --evaluation-periods 2 --threshold 40 \
  --comparison-operator LessThanOrEqualToThreshold \
  --dimensions "Name=AutoScalingGroupName,Value=my-asg" \
  --alarm-actions PolicyARN
```