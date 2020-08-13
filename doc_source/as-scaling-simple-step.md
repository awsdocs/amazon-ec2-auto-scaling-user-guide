# Step and simple scaling policies for Amazon EC2 Auto Scaling<a name="as-scaling-simple-step"></a>

With step scaling and simple scaling, you choose scaling metrics and threshold values for the CloudWatch alarms that trigger the scaling process\. You also define how your Auto Scaling group should be scaled when a threshold is in breach for a specified number of evaluation periods\. 

We strongly recommend that you use a target tracking scaling policy to scale on a metric like average CPU utilization or the `RequestCountPerTarget` metric from the Application Load Balancer\. Metrics that decrease when capacity increases and increase when capacity decreases can be used to proportionally scale out or in the number of instances using target tracking\. This helps ensure that Amazon EC2 Auto Scaling follows the demand curve for your applications closely\. For more information, see [Target tracking scaling policies](as-scaling-target-tracking.md)\. You still have the option to use step scaling as an additional policy for a more advanced configuration\. For example, you can configure a more aggressive response when demand reaches a certain level\.

**Contents**
+ [Differences between step scaling policies and simple scaling policies](#SimpleScaling)
+ [Step adjustments for step scaling](#as-scaling-steps)
+ [Scaling adjustment types](#as-scaling-adjustment)
+ [Instance warm\-up](#as-step-scaling-warmup)
+ [Create a CloudWatch alarm \(console\)](#policy-creating-alarm-console)
+ [Create step scaling policies \(console\)](#policy-creating-asg-console)
+ [Create scaling policies and CloudWatch alarms \(AWS CLI\)](#policy-creating-aws-cli)
  + [Step 1: Create an Auto Scaling group](#policy-creating-asg-aws-cli)
  + [Step 2: Create scaling policies](#policy-creating-policy-aws-cli)
    + [Step scaling policies](#step-scaling-policies-aws-cli)
    + [Simple scaling policies](#simple-scaling-policies-aws-cli)
  + [Step 3: Create CloudWatch alarms](#policy-creating-alarms-aws-cli)

## Differences between step scaling policies and simple scaling policies<a name="SimpleScaling"></a>

Step scaling policies and simple scaling policies are two of the dynamic scaling options available for you to use\. Both require you to create CloudWatch alarms for the scaling policies\. Both require you to specify the high and low thresholds for the alarms\. Both require you to define whether to add or remove instances, and how many, or set the group to an exact size\. 

The main difference between the policy types is the step adjustments that you get with step scaling policies\. When *step adjustments* are applied, and they increase or decrease the current capacity of your Auto Scaling group, the adjustments vary based on the size of the alarm breach\.

In most cases, step scaling policies are a better choice than simple scaling policies, even if you have only a single scaling adjustment\.

The main issue with simple scaling is that after a scaling activity is started, the policy must wait for the scaling activity or health check replacement to complete and the [cooldown period](Cooldown.md) to expire before responding to additional alarms\. Cooldown periods help to prevent the initiation of additional scaling activities before the effects of previous activities are visible\.

In contrast, with step scaling the policy can continue to respond to additional alarms, even while a scaling activity or health check replacement is in progress\. Therefore, all alarms that are breached are evaluated by Amazon EC2 Auto Scaling as it receives the alarm messages\. 

Amazon EC2 Auto Scaling originally supported only simple scaling policies\. If you created your scaling policy before target tracking and step policies were introduced, your policy is treated as a simple scaling policy\. 

## Step adjustments for step scaling<a name="as-scaling-steps"></a>

When you create a step scaling policy, you specify one or more step adjustments that automatically scale the number of instances dynamically based on the size of the alarm breach\. Each step adjustment specifies the following: 
+ A lower bound for the metric value
+ An upper bound for the metric value
+ The amount by which to scale, based on the scaling adjustment type 

CloudWatch aggregates metric data points based on the statistic for the metric that is associated with your CloudWatch alarm\. When the alarm is breached, the appropriate scaling policy is triggered\. Amazon EC2 Auto Scaling applies the aggregation type to the most recent metric data points from CloudWatch \(as opposed to the raw metric data\)\. It compares this aggregated metric value against the upper and lower bounds defined by the step adjustments to determine which step adjustment to perform\. 

You specify the upper and lower bounds relative to the breach threshold\. For example, let's say that you have an Auto Scaling group that has both a current capacity and a desired capacity of 10\. You have a CloudWatch alarm with a breach threshold of 50 percent and scale\-out and scale\-in policies\. You have a set of step adjustments with an adjustment type of `PercentChangeInCapacity` \(or **Percent of group** in the console\) for each policy: 


**Example: Step adjustments for scale\-out policy**  

| **Lower bound** | **Upper bound** | **Adjustment** | 
| --- | --- | --- | 
|  0  |  10  |  0  | 
|  10  |  20  |  10  | 
|  20  |  null  |  30  | 


**Example: Step adjustments for scale\-in policy**  

| **Lower bound** | **Upper bound** | **Adjustment** | 
| --- | --- | --- | 
|  \-10  |  0  |  0  | 
|  \-20  |  \-10  |  \-10  | 
|  null  |  \-20  |  \-30  | 

This creates the following scaling configuration\.

```
Metric value

-infinity          30%    40%          60%     70%             infinity
-----------------------------------------------------------------------
          -30%      | -10% | Unchanged  | +10%  |       +30%        
-----------------------------------------------------------------------
```

The following points summarize the behavior of the scaling configuration in relation to the desired and current capacity of the group:
+ The desired and current capacity is maintained while the aggregated metric value is greater than 40 and less than 60\.
+ If the metric value gets to 60, the desired capacity of the group increases by 1 instance, to 11 instances, based on the second step adjustment of the scale\-out policy \(add 10 percent of 10 instances\)\. After the new instance is running and its specified warm\-up time has expired, the current capacity of the group increases to 11 instances\. If the metric value rises to 70 even after this increase in capacity, the desired capacity of the group increases by another 3 instances, to 14 instances\. This is based on the third step adjustment of the scale\-out policy \(add 30 percent of 11 instances, 3\.3 instances, rounded down to 3 instances\)\.
+ If the metric value gets to 40, the desired capacity of the group decreases by 1 instance, to 13 instances, based on the second step adjustment of the scale\-in policy \(remove 10 percent of 14 instances, 1\.4 instances, rounded down to 1 instance\)\. If the metric value falls to 30 even after this decrease in capacity, the desired capacity of the group decreases by another 3 instances, to 10 instances\. This is based on the third step adjustment of the scale\-in policy \(remove 30 percent of 13 instances, 3\.9 instances, rounded down to 3 instances\)\.

When you specify the step adjustments for your scaling policy, note the following:
+ If you are using the AWS Management Console, you specify the upper and lower bounds as absolute values\. If you are using the AWS CLI or AWS SDKs, you specify the upper and lower bounds relative to the breach threshold\. 
+ The ranges of your step adjustments can't overlap or have a gap\.
+ Only one step adjustment can have a null lower bound \(negative infinity\)\. If one step adjustment has a negative lower bound, then there must be a step adjustment with a null lower bound\.
+ Only one step adjustment can have a null upper bound \(positive infinity\)\. If one step adjustment has a positive upper bound, then there must be a step adjustment with a null upper bound\.
+ The upper and lower bound can't be null in the same step adjustment\.
+ If the metric value is above the breach threshold, the lower bound is inclusive and the upper bound is exclusive\. If the metric value is below the breach threshold, the lower bound is exclusive and the upper bound is inclusive\.

## Scaling adjustment types<a name="as-scaling-adjustment"></a>

You can define a scaling policy that performs the optimal scaling action, based on the scaling adjustment type that you choose\. You can specify the adjustment type as a percentage of the current capacity of your Auto Scaling group, or in capacity units\. Normally a capacity unit means one instance, unless you are using the instance weighting feature\. 

Amazon EC2 Auto Scaling supports the following adjustment types for step scaling and simple scaling:
+ `ChangeInCapacity` — Increment or decrement the current capacity of the group by the specified value\. A positive value increases the capacity and a negative adjustment value decreases the capacity\. For example: If the current capacity of the group is 3 and the adjustment is 5, then when this policy is performed, we add 5 capacity units to the capacity for a total of 8 capacity units\. 
+ `ExactCapacity` — Change the current capacity of the group to the specified value\. Specify a positive value with this adjustment type\. For example: If the current capacity of the group is 3 and the adjustment is 5, then when this policy is performed, we change the capacity to 5 capacity units\. 
+ `PercentChangeInCapacity` — Increment or decrement the current capacity of the group by the specified percentage\. A positive value increases the capacity and a negative value decreases the capacity\. For example: If the current capacity is 10 and the adjustment is 10 percent, then when this policy is performed, we add 1 capacity unit to the capacity for a total of 11 capacity units\. 
**Note**  
If the resulting value is not an integer, it is rounded as follows:  
Values greater than 1 are rounded down\. For example, `12.7` is rounded to `12`\.
Values between 0 and 1 are rounded to 1\. For example, `.67` is rounded to `1`\.
Values between 0 and \-1 are rounded to \-1\. For example, `-.58` is rounded to `-1`\.
Values less than \-1 are rounded up\. For example, `-6.67` is rounded to `-6`\.

With `PercentChangeInCapacity`, you can also specify the minimum number of instances to scale using the `MinAdjustmentMagnitude` parameter\. For example, suppose that you create a policy that adds 25 percent and you specify a minimum increment of 2 instances\. If you have an Auto Scaling group with 4 instances and the scaling policy is executed, 25 percent of 4 is 1 instance\. However, because you specified a minimum increment of 2, there are 2 instances added\.

When [instance weighting](asg-instance-weighting.md) is used, the effect of setting the `MinAdjustmentMagnitude` parameter to a non\-zero value changes\. The value is in capacity units\. To set the minimum number of instances to scale, set this parameter to a value that is at least as large as your largest instance weight\.

If you are using instance weighting, keep in mind that the current capacity of your Auto Scaling group can exceed the desired capacity as needed\. If your absolute number to decrement, or the amount that the percentage says to decrement, is less than the difference between current and desired capacity, no scaling action is taken\. You must take these behaviors into account when you look at the outcome of a scaling policy when an alarm is triggered\. For example, suppose that the desired capacity is 30 and the current capacity is 32\. When the alarm is triggered, if the scaling policy decrements the desired capacity by 1, then no scaling action is taken\.

## Instance warm\-up<a name="as-step-scaling-warmup"></a>

If you are creating a step policy, you can specify the number of seconds that it takes for a newly launched instance to warm up\. Until its specified warm\-up time has expired, an instance is not counted toward the aggregated metrics of the Auto Scaling group\.

Using the example in the Step Adjustments section, suppose that the metric gets to 60, and then it gets to 62 while the new instance is still warming up\. The current capacity is still 10 instances, so 1 instance is added \(10 percent of 10 instances\)\. However, the desired capacity of the group is already 11 instances, so the scaling policy does not increase the desired capacity further\. If the metric gets to 70 while the new instance is still warming up, we should add 3 instances \(30 percent of 10 instances\)\. However, the desired capacity of the group is already 11, so we add only 2 instances, for a new desired capacity of 13 instances\.

While scaling out, we do not consider instances that are warming up as part of the current capacity of the group\. Therefore, multiple alarm breaches that fall in the range of the same step adjustment result in a single scaling activity\. This ensures that we don't add more instances than you need\.

While scaling in, we consider instances that are terminating as part of the current capacity of the group\. Therefore, we don't remove more instances from the Auto Scaling group than necessary\.

A scale\-in activity can't start while a scale\-out activity is in progress\.

## Create a CloudWatch alarm \(console\)<a name="policy-creating-alarm-console"></a>

You can use the following procedure to create the CloudWatch alarms that Amazon EC2 Auto Scaling uses to determine when to scale your Auto Scaling group\. Each CloudWatch alarm watches a single metric and sends messages to Amazon EC2 Auto Scaling when the metric breaches the alarm threshold\. 

**To create a CloudWatch alarm that monitors CPU utilization**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. If necessary, change the region\. From the navigation bar, select the region where your Auto Scaling group resides\.

1. In the navigation pane, choose **Alarms** and then choose **Create alarm**\.

1. <a name="step-create-alarm-start"></a>Choose **Select metric**\. 

1. On the **All metrics** tab, choose **EC2**, **By Auto Scaling Group**, and enter the Auto Scaling group's name in the search field\. Then, select `CPUUtilization` and choose **Select metric**\. The **Specify metric and conditions** page appears, showing a graph and other information about the metric\. 

1. For **Period**, choose the evaluation period for the alarm, for example, 1 minute\. When evaluating the alarm, each period is aggregated into one data point\. 
**Note**  
A shorter period creates a more sensitive alarm\.

1. Under **Conditions**, do the following:
   + For **Threshold type**, choose **Static**\.
   + For **Whenever `CPUUtilization` is**, specify whether you want the value of the metric to be greater than, greater than or equal to, less than, or less than or equal to the threshold to trigger the alarm\. Then, under **than**, enter the threshold value that you want to trigger the alarm\.

1. Under **Additional configuration**, do the following:
   + For **Datapoints to alarm**, enter the number of data points \(evaluation periods\) during which the metric value must meet the threshold conditions to trigger the alarm\. For example, two consecutive periods of 5 minutes would take 10 minutes to trigger the alarm\. 
   + For **Missing data treatment**, choose **Treat missing data as bad \(breaching threshold\)**\. For more information, see [Configuring how CloudWatch alarms treat missing data](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html#alarms-and-missing-data) in the *Amazon CloudWatch User Guide*\.

1. Choose **Next**\.

1. \(Optional\) Under **Notification**, you can choose or create the Amazon SNS topic you want to use to receive notifications\. Otherwise, you can remove the notification now and add one later as needed\.

1. Choose **Next**\.

1. Enter a name \(for example, `Step-Scaling-AlarmHigh-AddCapacity`\) and, optionally, a description for the alarm, and then choose **Next**\.

1. <a name="step-create-alarm-stop"></a>Choose **Create alarm**\.

**Example: CloudWatch alarm that triggers whenever CPU breaches the 80 percent threshold**

![\[CloudWatch Alarm\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/cw-console-create-alarm.png)

## Create step scaling policies \(console\)<a name="policy-creating-asg-console"></a>

The following procedure shows you how to use the Amazon EC2 Auto Scaling console to create two step scaling policies: a scale\-out policy that increases the capacity of the group by 30 percent, and a scale\-in policy that decreases the capacity of the group to two instances\.

While you are configuring your scaling policies, you can create the alarms at the same time\. Alternatively, you can use alarms that you created from the CloudWatch console, as described in the previous section\.

**To create a step scaling policy for scale out**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\. 

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. Verify that the minimum and maximum size limits are appropriately set\. For example, if your group is already at its maximum size, you need to specify a new maximum in order to scale out\. Amazon EC2 Auto Scaling does not scale your group below the minimum capacity or above the maximum capacity\. To update your group, on the **Details** tab, change the current settings for minimum and maximum capacity\. 

1. On the **Automatic scaling** tab, in **Scaling policies**, choose **Add policy**\. \(Old console: The **Scaling Policies** tab is where you can add the policy\.\) 

1. To define a policy for scale out \(increase capacity\), do the following:

   1. For **Policy type**, choose **Step scaling**\. \(Old console: Choose **Create a scaling policy with steps**\.\) 

   1. Specify a name for the policy\. 

   1. For **CloudWatch alarm**, choose your alarm\. If you haven't already created an alarm, choose **Create a CloudWatch alarm** and complete [Step 4](#step-create-alarm-start) through [Step 13](#step-create-alarm-stop) to create an alarm that monitors CPU utilization\. Set the alarm threshold to greater than or equal to 80 percent\. 

   1. Specify the change in the current group size that this policy will make when executed using **Take the action**\. You can add a specific number of instances or a percentage of the existing group size, or set the group to an exact size\. 

      For example, choose `Add`, enter `30` in the next field, and then choose `percent of group`\. By default, the lower bound for this step adjustment is the alarm threshold and the upper bound is positive infinity\. 

   1. To add another step, choose **Add step** and then define the amount by which to scale and the lower and upper bounds of the step relative to the alarm threshold\. 

   1. To set a minimum number of instances to scale, update the number field in **Add capacity units in increments of at least** `1` **capacity units**\. 

   1. Specify an instance warm\-up value for **Instances need**, which allows you to control the amount of time until a newly launched instance can contribute to the CloudWatch metrics\. 

1. Choose **Create**\.

**To create a step scaling policy for scale in**

1. Choose **Add policy** to continue where you left off after creating a policy for scale out\.

1. To define a policy for scale in \(decrease capacity\), do the following:

   1. For **Policy type**, choose **Step scaling**\. \(Old console: Choose **Create a scaling policy with steps**\.\) 

   1. Specify a name for the policy\. 

   1. For **CloudWatch alarm**, choose your alarm\. If you haven't already created an alarm, choose **Create a CloudWatch alarm** and complete [Step 4](#step-create-alarm-start) through [Step 13](#step-create-alarm-stop) to create an alarm that monitors CPU utilization\. Set the alarm threshold to less than or equal to 40 percent\. 

   1. Specify the change in the current group size that this policy will make when executed using **Take the action**\. You can remove a specific number of instances or a percentage of the existing group size, or set the group to an exact size\. 

      For example, choose `Remove`, enter `2` in the next field, and then choose `capacity units`\. By default, the upper bound for this step adjustment is the alarm threshold and the lower bound is negative infinity\. 

   1. To add another step, choose **Add step** and then define the amount by which to scale and the lower and upper bounds of the step relative to the alarm threshold\. 

1. Choose **Create**\.

## Create scaling policies and CloudWatch alarms \(AWS CLI\)<a name="policy-creating-aws-cli"></a>

Use the AWS CLI as follows to configure step or simple scaling policies for your Auto Scaling group\.

**Topics**
+ [Step 1: Create an Auto Scaling group](#policy-creating-asg-aws-cli)
+ [Step 2: Create scaling policies](#policy-creating-policy-aws-cli)
+ [Step 3: Create CloudWatch alarms](#policy-creating-alarms-aws-cli)

### Step 1: Create an Auto Scaling group<a name="policy-creating-asg-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group named `my-asg` using the launch configuration `my-launch-config`\. If you don't have a launch configuration that you'd like to use, you can create one by calling [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html)\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --max-size 5 --min-size 1
```

### Step 2: Create scaling policies<a name="policy-creating-policy-aws-cli"></a>

You can create step or simple scaling policies that tell the Auto Scaling group what to do when the load on the application changes\.

#### Step scaling policies<a name="step-scaling-policies-aws-cli"></a>

**Example: my\-step\-scale\-out\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a step scaling policy named `my-step-scale-out-policy`, with an adjustment type of `PercentChangeInCapacity` that increases the capacity of the group based on the following step adjustments \(assuming a CloudWatch alarm threshold of 60 percent\):
+ Increase the instance count by 10 percent when the value of the metric is greater than or equal to 70 percent but less than 80 percent 
+ Increase the instance count by 20 percent when the value of the metric is greater than or equal to 80 percent but less than 90 percent
+ Increase the instance count by 30 percent when the value of the metric is greater than or equal to 90 percent

```
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg  \
  --policy-name my-step-scale-out-policy \
  --policy-type StepScaling \
  --adjustment-type PercentChangeInCapacity \
  --metric-aggregation-type Average \
  --step-adjustments MetricIntervalLowerBound=10.0,MetricIntervalUpperBound=20.0,ScalingAdjustment=10 \
                     MetricIntervalLowerBound=20.0,MetricIntervalUpperBound=30.0,ScalingAdjustment=20 \
                     MetricIntervalLowerBound=30.0,ScalingAdjustment=30 \
  --min-adjustment-magnitude 1
```

Record the policy's Amazon Resource Name \(ARN\)\. You need it to create a CloudWatch alarm for the policy\. 

```
{
    "PolicyARN": "arn:aws:autoscaling:region:123456789012:scalingPolicy:4ee9e543-86b5-4121-b53b-aa4c23b5bbcc:autoScalingGroupName/my-asg:policyName/my-step-scale-in-policy
}
```

**Example: my\-step\-scale\-in\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a step scaling policy named `my-step-scale-in-policy`, with an adjustment type of `ChangeInCapacity` that decreases the capacity of the group by 2 instances\.

```
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg  \
  --policy-name my-step-scale-in-policy \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --step-adjustments MetricIntervalUpperBound=0.0,ScalingAdjustment=-2
```

Record the policy's Amazon Resource Name \(ARN\)\. You need it to create a CloudWatch alarm for the policy\. 

```
{
    "PolicyARN": "arn:aws:autoscaling:region:123456789012:scalingPolicy:ac542982-cbeb-4294-891c-a5a941dfa787:autoScalingGroupName/my-asg:policyName/my-step-scale-out-policy
}
```

#### Simple scaling policies<a name="simple-scaling-policies-aws-cli"></a>

Alternatively, you can create simple scaling policies by using the following CLI commands instead of the preceding CLI commands\. Keep in mind that a cooldown period will be in place due to the use of simple scaling policies\. 

**Example: my\-simple\-scale\-out\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a simple scaling policy named `my-simple-scale-out-policy`, with an adjustment type of `PercentChangeInCapacity` that increases the capacity of the group by 30 percent\.

```
aws autoscaling put-scaling-policy --policy-name my-simple-scale-out-policy \
  --auto-scaling-group-name my-asg --scaling-adjustment 30 \
  --adjustment-type PercentChangeInCapacity
```

Record the policy's Amazon Resource Name \(ARN\)\. You need it to create a CloudWatch alarm for the policy\. 

**Example: my\-simple\-scale\-in\-policy**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command to create a simple scaling policy named `my-simple-scale-in-policy`, with an adjustment type of `ChangeInCapacity` that decreases the capacity of the group by one instance\.

```
aws autoscaling put-scaling-policy --policy-name my-simple-scale-in-policy \
  --auto-scaling-group-name my-asg --scaling-adjustment -1 \
  --adjustment-type ChangeInCapacity --cooldown 180
```

Record the policy's Amazon Resource Name \(ARN\)\. You need it to create a CloudWatch alarm for the policy\. 

### Step 3: Create CloudWatch alarms<a name="policy-creating-alarms-aws-cli"></a>

In step 2, you created scaling policies that provided instructions to the Auto Scaling group about how to scale in and scale out when the conditions that you specify change\. In this step, you create alarms by identifying the metrics to watch, defining the conditions for scaling, and then associating the alarms with the scaling policies\. 

**Example: AddCapacity**  
Use the following CloudWatch [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to create an alarm that increases the size of the Auto Scaling group based on an average CPU threshold value of 60 percent for at least two consecutive evaluation periods of two minutes\. To use your own custom metric, specify its name in `--metric-name` and its namespace in `--namespace`\.

```
aws cloudwatch put-metric-alarm --alarm-name Step-Scaling-AlarmHigh-AddCapacity \
  --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average \
  --period 120 --evaluation-periods 2 --threshold 60 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --dimensions "Name=AutoScalingGroupName,Value=my-asg" \
  --alarm-actions PolicyARN
```

**Example: RemoveCapacity**  
Use the following CloudWatch [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to create an alarm that decreases the size of the Auto Scaling group based on average CPU threshold value of 40 percent for at least two consecutive evaluation periods of two minutes\. To use your own custom metric, specify its name in `--metric-name` and its namespace in `--namespace`\.

```
aws cloudwatch put-metric-alarm --alarm-name Step-Scaling-AlarmLow-RemoveCapacity \
  --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average \
  --period 120 --evaluation-periods 2 --threshold 40 \
  --comparison-operator LessThanOrEqualToThreshold \
  --dimensions "Name=AutoScalingGroupName,Value=my-asg" \
  --alarm-actions PolicyARN
```