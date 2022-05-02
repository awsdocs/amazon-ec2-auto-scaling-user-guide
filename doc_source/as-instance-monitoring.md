# Monitoring CloudWatch metrics for your Auto Scaling groups and instances<a name="as-instance-monitoring"></a>

*Metrics* are the fundamental concept in CloudWatch\. A metric represents a time\-ordered set of data points that are published to CloudWatch\. Think of a metric as a variable to monitor, and the data points as representing the values of that variable over time\. You can use these metrics to verify that your system is performing as expected\. 

Amazon EC2 Auto Scaling publishes data points to CloudWatch about your Auto Scaling groups\. The metrics are available at one\-minute granularity at no additional charge, but you must enable them\. By doing this, you get continuous visibility into the operations of your Auto Scaling groups so that you can quickly respond to changes in your workloads\. You can enable and disable these metrics using the AWS Management Console, AWS CLI, or an SDK\. 

Amazon EC2 publishes data points to CloudWatch that describe your Auto Scaling instances\. The interval for Amazon EC2 instance monitoring is configurable\. You can choose between one\-minute and five\-minute granularity\. For more information, see [Configuring monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\.

**Contents**
+ [Enable Auto Scaling group metrics \(console\)](#as-enable-group-metrics)
+ [Enable Auto Scaling group metrics \(AWS CLI\)](#as-enable-group-metrics-cli)
+ [Available metrics and dimensions](#available-cloudwatch-metrics)
  + [Auto Scaling group metrics](#as-group-metrics)
  + [Dimensions for Auto Scaling group metrics](#as-group-metric-dimensions)
+ [Working with Amazon CloudWatch](#cloudwatch-working)
  + [View CloudWatch metrics](#cloudwatch-view-metrics)
  + [Create Amazon CloudWatch alarms](#cloudwatch-create-alarm)
+ [Configuring monitoring for Auto Scaling instances](enable-as-instance-metrics.md)
  + [Enable detailed monitoring \(console\)](enable-as-instance-metrics.md#enable-detailed-monitoring-console)
  + [Enable detailed monitoring \(AWS CLI\)](enable-as-instance-metrics.md#enable-detailed-monitoring-cli)
  + [Switching between basic and detailed monitoring](enable-as-instance-metrics.md#change-monitoring)
+ [Viewing monitoring graphs in the Amazon EC2 Auto Scaling console](viewing-monitoring-graphs.md)
  + [Graph metrics for your Auto Scaling groups](viewing-monitoring-graphs.md#graph-metrics)

## Enable Auto Scaling group metrics \(console\)<a name="as-enable-group-metrics"></a>

When you enable Auto Scaling group metrics, your Auto Scaling group sends sampled data to CloudWatch every minute\. There is no charge for enabling these metrics\.

**To enable group metrics**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Monitoring** tab, select the **Auto Scaling group metrics collection**, **Enable** check box located at the top of the page under **Auto Scaling**\. 

**To disable group metrics**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select your Auto Scaling group\.

1. On the **Monitoring** tab, clear the **Auto Scaling group metrics collection**, **Enable** check box\. 

## Enable Auto Scaling group metrics \(AWS CLI\)<a name="as-enable-group-metrics-cli"></a>

**To enable group metrics**  
Enable one or more group metrics using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/enable-metrics-collection.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/enable-metrics-collection.html) command\. For example, the following command enables the GroupDesiredCapacity metric\.

```
aws autoscaling enable-metrics-collection --auto-scaling-group-name my-asg \
--metrics GroupDesiredCapacity --granularity "1Minute"
```

If you omit the `--metrics` option, all metrics are enabled\.

```
aws autoscaling enable-metrics-collection --auto-scaling-group-name my-asg \
--granularity "1Minute"
```

**To disable group metrics**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/disable-metrics-collection.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/disable-metrics-collection.html) command\. For example, the following command disables all Auto Scaling group metrics\.

```
aws autoscaling disable-metrics-collection --auto-scaling-group-name my-asg
```

## Available metrics and dimensions<a name="available-cloudwatch-metrics"></a>

When enabled, the Auto Scaling group metrics and dimensions that Amazon EC2 Auto Scaling sends to CloudWatch are listed below\.

**Note**  
CloudWatch group metrics are delivered on a best\-effort basis\. Data is not backfilled to fill gaps in group metric history in rare cases where CloudWatch is experiencing a service disruption\.

### Auto Scaling group metrics<a name="as-group-metrics"></a>

The `AWS/AutoScaling` namespace includes the following metrics\.


| Metric | Description | 
| --- | --- | 
|  GroupMinSize  |  The minimum size of the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupMaxSize  |  The maximum size of the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupDesiredCapacity  |  The number of instances that the Auto Scaling group attempts to maintain\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupInServiceInstances  |  The number of instances that are running as part of the Auto Scaling group\. This metric does not include instances that are pending or terminating\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupPendingInstances  |  The number of instances that are pending\. A pending instance is not yet in service\. This metric does not include instances that are in service or terminating\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupStandbyInstances  |  The number of instances that are in a `Standby` state\. Instances in this state are still running but are not actively in service\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupTerminatingInstances  |  The number of instances that are in the process of terminating\. This metric does not include instances that are in service or pending\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupTotalInstances  |  The total number of instances in the Auto Scaling group\. This metric identifies the number of instances that are in service, pending, and terminating\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 

The `AWS/AutoScaling` namespace includes the following metrics for Auto Scaling groups that use the [instance weighting](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md) feature\. If instance weighting is not applied, then the following metrics are populated, but are equal to the metrics that are defined in the preceding table\.


| Metric | Description | 
| --- | --- | 
|  GroupInServiceCapacity  |  The number of capacity units that are running as part of the Auto Scaling group\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupPendingCapacity  |  The number of capacity units that are pending\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupStandbyCapacity  |  The number of capacity units that are in a `Standby` state\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupTerminatingCapacity  |  The number of capacity units that are in the process of terminating\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupTotalCapacity  |  The total number of capacity units in the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 

The `AWS/AutoScaling` namespace includes the following metrics for Auto Scaling groups that use the [warm pools](ec2-auto-scaling-warm-pools.md) feature\.


| Metric | Description | 
| --- | --- | 
|  WarmPoolMinSize  |  The minimum size of the warm pool\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  WarmPoolDesiredCapacity  |  The amount of capacity that Amazon EC2 Auto Scaling attempts to maintain in the warm pool\.  This is equivalent to the maximum size of the Auto Scaling group minus its desired capacity, or, if set, as the maximum prepared capacity of the Auto Scaling group minus its desired capacity\. However, when the minimum size of the warm pool is equal to or greater than the difference between the maximum size \(or, if set, the maximum prepared capacity\) and the desired capacity of the Auto Scaling group, then the warm pool desired capacity will be equivalent to the `WarmPoolMinSize`\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  WarmPoolPendingCapacity  |  The amount of capacity in the warm pool that is pending\. This metric does not include instances that are running, stopped, or terminating\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  WarmPoolTerminatingCapacity  |  The amount of capacity in the warm pool that is in the process of terminating\. This metric does not include instances that are running, stopped, or pending\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  WarmPoolWarmedCapacity  |  The amount of capacity available to enter the Auto Scaling group during scale out\. This metric does not include instances that are pending or terminating\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  WarmPoolTotalCapacity  |  The total capacity of the warm pool, including instances that are running, stopped, pending, or terminating\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupAndWarmPoolDesiredCapacity  |  The desired capacity of the Auto Scaling group and the warm pool combined\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupAndWarmPoolTotalCapacity  |  The total capacity of the Auto Scaling group and the warm pool combined\. This includes instances that are running, stopped, pending, terminating, or in service\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 

### Auto Scaling group dimensions<a name="as-group-metric-dimensions"></a>

| Dimension | Description | 
| --- | --- | 
| AutoScalingGroupName | Filter metric for a specific Auto Scaling group by name\. |


## Working with Amazon CloudWatch<a name="cloudwatch-working"></a>

**Contents**
+ [View CloudWatch metrics](#cloudwatch-view-metrics)
+ [Create Amazon CloudWatch alarms](#cloudwatch-create-alarm)

### View CloudWatch metrics<a name="cloudwatch-view-metrics"></a>

You can view your Auto Scaling group metrics using the CloudWatch console and the command line tools\. 

**To view metrics using the CloudWatch console**  
For more information, see [Aggregating statistics by Auto Scaling group](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/GetMetricAutoScalingGroup.html)\.

**To view CloudWatch metrics \(AWS CLI\)**  
To view all metrics for all your Auto Scaling groups, use the following [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/list-metrics.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/list-metrics.html) command\.

```
aws cloudwatch list-metrics --namespace "AWS/AutoScaling"
```

To view the metrics for a single Auto Scaling group, specify the `AutoScalingGroupName` dimension as follows\.

```
aws cloudwatch list-metrics --namespace "AWS/AutoScaling" --dimensions Name=AutoScalingGroupName,Value=my-asg
```

To view a single metric for all your Auto Scaling groups, specify the name of the metric as follows\.

```
aws cloudwatch list-metrics --namespace "AWS/AutoScaling" --metric-name GroupDesiredCapacity
```

### Create Amazon CloudWatch alarms<a name="cloudwatch-create-alarm"></a>

One purpose for monitoring metrics is to verify that your application is performing as expected\. In Amazon CloudWatch, you can create an alarm that sends a notification when the value of a certain metric is beyond what you consider an acceptable threshold\. 

Start by identifying the metric to monitor\. For example, you can configure an alarm to watch over the average CPU utilization of the EC2 instances in your Auto Scaling group\. The action can be a notification that is sent to you when the average CPU utilization of the group's instances breaches the threshold that you specified for the consecutive periods you specified\. For example, if the metric stays at or above 70 percent for 4 consecutive periods of 1 minute each\. 

For more information, see [Using Amazon CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) in the *Amazon CloudWatch User Guide*\.

**To create a CloudWatch alarm for your Auto Scaling group**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. If necessary, change the region\. From the navigation bar, select the region where your Auto Scaling group resides\.

1. On the navigation pane, choose **Alarms** and then choose **Create alarm**\.

1. Choose **Select metric**\. 

1. On the **All metrics** tab, select a metric as follows:
   + To display only the metrics reported for your Auto Scaling groups, choose **EC2**, and then choose **By Auto Scaling Group**\. To view the metrics for a single Auto Scaling group, type its name in the search field\. 
   + Select the row that contains the metric for the Auto Scaling group that you want to create an alarm on\. 
   + Choose **Select metric**\. The **Specify metric and conditions** page appears, showing a graph and other information about the metric\. 

1. For **Period**, choose the evaluation period for the alarm, for example, 1 minute\. When evaluating the alarm, each period is aggregated into one data point\. 
**Note**  
A shorter period creates a more sensitive alarm\.

1. Under **Conditions**, do the following:
   + For **Threshold type**, choose **Static**\.
   + For **Whenever `metric` is**, specify whether you want the value of the metric to be greater than, greater than or equal to, less than, or less than or equal to the threshold to trigger the alarm\. Then, under **than**, enter the threshold value that you want to trigger the alarm\.

1. Under **Additional configuration**, do the following:
   + For **Datapoints to alarm**, enter the number of data points \(evaluation periods\) during which the metric value must meet the threshold conditions to trigger the alarm\. For example, two consecutive periods of 5 minutes would take 10 minutes to trigger the alarm\.
   + For **Missing data treatment**, choose what you want the alarm to do if some data is missing\. For more information, see [Configuring how CloudWatch alarms treat missing data](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html#alarms-and-missing-data) in the *Amazon CloudWatch User Guide*\.

1. Choose **Next**\.

1. Under **Notification**, you can choose or create the Amazon SNS topic you want to use to receive notifications\. Otherwise, you can remove the notification now and add one later when you are ready\.

1. Choose **Next**\.

1. Enter a name and, optionally, a description for the alarm, and then choose **Next**\.

1. Choose **Create Alarm**\.
