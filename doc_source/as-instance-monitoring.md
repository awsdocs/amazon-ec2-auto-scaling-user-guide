# Monitoring CloudWatch metrics for your Auto Scaling groups and instances<a name="as-instance-monitoring"></a>

*Metrics* are the fundamental concept in CloudWatch\. A metric represents a time\-ordered set of data points that are published to CloudWatch\. Think of a metric as a variable to monitor, and the data points as representing the values of that variable over time\. You can use these metrics to verify that your system is performing as expected\. 

Amazon EC2 Auto Scaling publishes data points to CloudWatch about your Auto Scaling groups\. The metrics are available at 1\-minute granularity at no additional charge, but you must enable them\. By doing this, you get continuous visibility into the operations of your Auto Scaling groups so that you can quickly respond to changes in your workloads\. The following sections guide you through enabling them\.

Amazon EC2 publishes data points to CloudWatch that describe your Auto Scaling instances\. The interval for Amazon EC2 instance monitoring is configurable\. You can choose between 1\-minute and 5\-minute granularity\. 

**Contents**
+ [Enabling Auto Scaling group metrics](#as-enable-group-metrics)
+ [Available metrics and dimensions](#available-cloudwatch-metrics)
  + [Auto Scaling group metrics](#as-group-metrics)
  + [Dimensions for Auto Scaling group metrics](#as-group-metric-dimensions)
+ [Viewing graphed metrics for your Auto Scaling groups and instances](#as-view-group-metrics)
+ [Working with Amazon CloudWatch](#cloudwatch-working)
  + [Viewing CloudWatch metrics](#cloudwatch-view-metrics)
  + [Creating Amazon CloudWatch alarms](#cloudwatch-create-alarm)
+ [Configuring monitoring for Auto Scaling instances](enable-as-instance-metrics.md)

## Enabling Auto Scaling group metrics<a name="as-enable-group-metrics"></a>

When you enable Auto Scaling group metrics, your Auto Scaling group sends sampled data to CloudWatch every minute\. There is no charge for enabling these metrics\.

You can enable and disable Auto Scaling group metrics using the AWS Management Console, AWS CLI, or AWS SDKs\. 

**To enable group metrics \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Monitoring** tab, select the **Auto Scaling group metrics collection**, **Enable** check box located at the top of the page under **Auto Scaling**\. 

   \(Old console: On the **Monitoring** tab, for **Auto Scaling Metrics**, choose **Enable Group Metrics Collection**\. If you don't see this option, select **Auto Scaling** for **Display**\.\)

**To disable group metrics \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Monitoring** tab, clear the **Auto Scaling group metrics collection**, **Enable** check box\. 

   \(Old console: On the **Monitoring** tab, for **Auto Scaling Metrics**, choose **Disable Group Metrics Collection**\.\)

**To enable group metrics \(AWS CLI\)**  
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

**To disable group metrics \(AWS CLI\)**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/disable-metrics-collection.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/disable-metrics-collection.html) command\. For example, the following command disables all Auto Scaling group metrics\.

```
aws autoscaling disable-metrics-collection --auto-scaling-group-name my-asg
```

## Available metrics and dimensions<a name="available-cloudwatch-metrics"></a>

### Auto Scaling group metrics<a name="as-group-metrics"></a>

The `AWS/AutoScaling` namespace includes the following metrics\.


| Metric | Description | 
| --- | --- | 
| GroupMinSize |  The minimum size of the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupMaxSize |  The maximum size of the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupDesiredCapacity |  The number of instances that the Auto Scaling group attempts to maintain\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupInServiceInstances |  The number of instances that are running as part of the Auto Scaling group\. This metric does not include instances that are pending or terminating\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupPendingInstances |  The number of instances that are pending\. A pending instance is not yet in service\. This metric does not include instances that are in service or terminating\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupStandbyInstances |  The number of instances that are in a `Standby` state\. Instances in this state are still running but are not actively in service\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupTerminatingInstances |  The number of instances that are in the process of terminating\. This metric does not include instances that are in service or pending\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupTotalInstances |  The total number of instances in the Auto Scaling group\. This metric identifies the number of instances that are in service, pending, and terminating\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 

The `AWS/AutoScaling` namespace includes the following metrics for Auto Scaling groups that use the [instance weighting](asg-instance-weighting.md) feature\. If instance weighting is not applied, then the following metrics are populated, but are equal to the metrics that are defined in the preceding table\.


| Metric | Description | 
| --- | --- | 
| GroupInServiceCapacity | The number of capacity units that are running as part of the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\. | 
| GroupPendingCapacity |  The number of capacity units that are pending\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupStandbyCapacity |  The number of capacity units that are in a `Standby` state\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupTerminatingCapacity |  The number of capacity units that are in the process of terminating\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupTotalCapacity |  The total number of capacity units in the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 

### Dimensions for Auto Scaling group metrics<a name="as-group-metric-dimensions"></a>

To filter the metrics for your Auto Scaling group by group name, use the `AutoScalingGroupName` dimension\.

## Viewing graphed metrics for your Auto Scaling groups and instances<a name="as-view-group-metrics"></a>

After you create an Auto Scaling group, you can open the group and view a series of monitoring graphs on the **Monitoring** tab\. Each graph is based on one of the available CloudWatch metrics for your Auto Scaling groups and instances\. The monitoring graphs show data points for Auto Scaling group metrics if the metrics are enabled\. 

The following graphed metrics are available for groups:
+ **Minimum Group Size** — `GroupMinSize`
+ **Maximum Group Size** — `GroupMaxSize`
+ **Desired Capacity** — `GroupDesiredCapacity`
+ **In Service Instances** — `GroupInServiceInstances`
+ **Pending Instances** — `GroupPendingInstances`
+ **Standby Instances** — `GroupStandbyInstances`
+ **Terminating Instances** — `GroupTerminatingInstances`
+ **Total Instances** — `GroupTotalInstances`

The following graphed metrics are available for groups where instances have weights that define how many units each instance contributes to the desired capacity of the group:
+ **In Service Capacity Units** — `GroupInServiceCapacity`
+ **Pending Capacity Units** — `GroupPendingCapacity`
+ **Standby Capacity Units** — `GroupStandbyCapacity`
+ **Terminating Capacity Units** — `GroupTerminatingCapacity`
+ **Total Capacity Units** — `GroupTotalCapacity`

The following metrics are available for instances:
+ **CPU Utilization** — `CPUUtilization`
+ **Disk Reads** — `DiskReadBytes`
+ **Disk Read Operations** — `DiskReadOps`
+ **Disk Writes** — `DiskWriteBytes`
+ **Disk Write Operations** — `DiskWriteOps`
+ **Network In** — `NetworkIn`
+ **Network Out** — `NetworkOut`
+ **Status Check Failed \(Any\)** — `StatusCheckFailed`
+ **Status Check Failed \(Instance\)** — `StatusCheckFailed_Instance`
+ **Status Check Failed \(System\)** — `StatusCheckFailed_System`

For more information about the Amazon EC2 metrics and the data they provide to the graphs, see [List the available CloudWatch metrics for your instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

## Working with Amazon CloudWatch<a name="cloudwatch-working"></a>

**Contents**
+ [Viewing CloudWatch metrics](#cloudwatch-view-metrics)
+ [Creating Amazon CloudWatch alarms](#cloudwatch-create-alarm)

### Viewing CloudWatch metrics<a name="cloudwatch-view-metrics"></a>

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

### Creating Amazon CloudWatch alarms<a name="cloudwatch-create-alarm"></a>

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