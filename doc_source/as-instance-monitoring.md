# Monitoring Your Auto Scaling Groups and Instances Using Amazon CloudWatch<a name="as-instance-monitoring"></a>

*Metrics* are the fundamental concept in CloudWatch\. A metric represents a time\-ordered set of data points that are published to CloudWatch\. Think of a metric as a variable to monitor, and the data points as representing the values of that variable over time\. You can use these metrics to verify that your system is performing as expected\. 

Amazon EC2 Auto Scaling publishes data points to CloudWatch about your Auto Scaling groups\. The metrics are available at 1\-minute granularity at no additional charge, but you must enable them\. By doing this, you get continuous visibility into the operations of your Auto Scaling groups so that you can quickly respond to changes in your workloads\. The following sections guide you through enabling them\.

Amazon EC2 publishes data points to CloudWatch that describe your Auto Scaling instances\. The interval for Amazon EC2 instance monitoring is configurable\. You can choose between 1\-minute and 5\-minute granularity, as described in the sections that follow\.

**Note**  
Amazon EC2 monitoring is available for any EC2 instance, not just those in an Auto Scaling group\. For more information, see [Monitoring Your Instances Using CloudWatch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**Topics**
+ [Auto Scaling Group Metrics](#as-group-metrics)
+ [Dimensions for Auto Scaling Group Metrics](#as-group-metric-dimensions)
+ [Enable Auto Scaling Group Metrics](#as-enable-group-metrics)
+ [Configure Monitoring for Auto Scaling Instances](#enable-as-instance-metrics)
+ [View CloudWatch Metrics](#as-view-group-metrics)
+ [Create Amazon CloudWatch Alarms](#CloudWatchAlarm)

## Auto Scaling Group Metrics<a name="as-group-metrics"></a>

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

The `AWS/AutoScaling` namespace includes the following metrics for your groups that use the [instance weighting](asg-instance-weighting.md) feature\. If instance weighting is not applied, then the following metrics are equal to the corresponding metrics defined in the preceding table\.


| Metric | Description | 
| --- | --- | 
| GroupInServiceCapacity | The number of capacity units that are running as part of the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\. | 
| GroupPendingCapacity |  The number of capacity units that are pending\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupStandbyCapacity |  The number of capacity units that are in a `Standby` state\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupTerminatingCapacity |  The number of capacity units that are in the process of terminating\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
| GroupTotalCapacity |  The total number of capacity units in the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 

## Dimensions for Auto Scaling Group Metrics<a name="as-group-metric-dimensions"></a>

To filter the metrics for your Auto Scaling group by group name, use the `AutoScalingGroupName` dimension\.

## Enable Auto Scaling Group Metrics<a name="as-enable-group-metrics"></a>

When you enable Auto Scaling group metrics, your Auto Scaling group sends sampled data to CloudWatch every minute\. There is no charge for enabling these metrics\.

**To enable group metrics \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

1. On the **Monitoring** tab, for **Auto Scaling Metrics**, choose **Enable Group Metrics Collection**\. If you don't see this option, select **Auto Scaling** for **Display**\.  
![\[Enable Auto Scaling group metrics collection.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/monitoring_group_metrics_enable.png)

**To disable group metrics \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Monitoring** tab, for **Auto Scaling Metrics**, choose **Disable Group Metrics Collection**\. If you don't see this option, select **Auto Scaling** for **Display**\.  
![\[Disable Auto Scaling group metrics collection.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/monitoring_group_metrics_disable.png)

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

## Configure Monitoring for Auto Scaling Instances<a name="enable-as-instance-metrics"></a>

You configure monitoring for EC2 instances using a launch configuration or template\. Monitoring is enabled whenever an instance is launched, either basic monitoring \(5\-minute granularity\) or detailed monitoring \(1\-minute granularity\)\. For detailed monitoring, additional charges apply\. For more information, see [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)\.

**Note**  
By default, basic monitoring is enabled when you create a launch template or when you use the AWS Management Console to create a launch configuration\. Detailed monitoring is enabled by default when you create a launch configuration using the AWS CLI or an SDK\. 

To change the type of monitoring enabled on new EC2 instances, update the launch template or update the Auto Scaling group to use a new launch configuration\. Existing instances continue to use the previously enabled monitoring type\. To update all instances, terminate them so that they are replaced by your Auto Scaling group or update instances individually using [https://docs.aws.amazon.com/cli/latest/reference/ec2/monitor-instances.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/monitor-instances.html) and [https://docs.aws.amazon.com/cli/latest/reference/ec2/unmonitor-instances.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/unmonitor-instances.html)\.

If you have CloudWatch alarms associated with your Auto Scaling group, use the [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to update each alarm\. Make each period match the monitoring type \(300 seconds for basic monitoring and 60 seconds for detailed monitoring\)\. If you change from detailed monitoring to basic monitoring but do not update your alarms to match the five\-minute period, they continue to check for statistics every minute\. They might find no data available for as many as four out of every five periods\.

**To configure CloudWatch monitoring \(console\)**  
When you create the launch configuration using the AWS Management Console, on the **Configure Details** page, select **Enable CloudWatch detailed monitoring**\. Otherwise, basic monitoring is enabled\. For more information, see [Creating a Launch Configuration](create-launch-config.md)\.

To enable detailed monitoring for a launch template using the AWS Management Console, in the **Advanced Details** section, for **Monitoring**, choose **Enable**\. Otherwise, basic monitoring is enabled\. For more information, see [Creating a Launch Template for an Auto Scaling Group](create-launch-template.md)\.

**To configure CloudWatch monitoring \(AWS CLI\)**  
For launch configurations, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command with the `--instance-monitoring` option\. Set this option to `true` to enable detailed monitoring or `false` to enable basic monitoring\.

```
--instance-monitoring Enabled=true
```

For launch templates, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command and pass a JSON file that contains the information for creating the launch template\. Set the monitoring attribute to `"Monitoring":{"Enabled":true}` to enable detailed monitoring or `"Monitoring":{"Enabled":false}` to enable basic monitoring\. 

## View CloudWatch Metrics<a name="as-view-group-metrics"></a>

You can view the CloudWatch metrics for your Auto Scaling groups and instances using the Amazon EC2 console\. These metrics are displayed as monitoring graphs\.

Alternatively, you can view these metrics using the CloudWatch console\.

**To view metrics using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. Choose the **Monitoring** tab\.

1. \(Optional\) To filter the results by time, select a time range from **Showing data for**\.

1. To view the metrics for your groups, for **Display**, choose **Auto Scaling**\. To get a larger view of a single metric, select its graph\. The following metrics are available for groups:
   + Minimum Group Size — `GroupMinSize`
   + Maximum Group Size — `GroupMaxSize`
   + Desired Capacity — `GroupDesiredCapacity`
   + In Service Instances — `GroupInServiceInstances`
   + Pending Instances — `GroupPendingInstances`
   + Standby Instances — `GroupStandbyInstances`
   + Terminating Instances — `GroupTerminatingInstances`
   + Total Instances — `GroupTotalInstances`

1. To view metrics for your instances, for **Display**, choose **EC2**\. To get a larger view of a single metric, select its graph\. The following metrics are available for instances:
   + CPU Utilization — `CPUUtilization`
   + Disk Reads — `DiskReadBytes`
   + Disk Read Operations — `DiskReadOps`
   + Disk Writes — `DiskWriteBytes`
   + Disk Write Operations — `DiskWriteOps`
   + Network In — `NetworkIn`
   + Network Out — `NetworkOut`
   + Status Check Failed \(Any\) — `StatusCheckFailed`
   + Status Check Failed \(Instance\) — `StatusCheckFailed_Instance`
   + Status Check Failed \(System\) — `StatusCheckFailed_System`

**To view metrics using the CloudWatch console**  
For more information, see [Aggregating Statistics by Auto Scaling Group](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/GetMetricAutoScalingGroup.html)\.

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

## Create Amazon CloudWatch Alarms<a name="CloudWatchAlarm"></a>

One purpose for monitoring metrics is to verify that your application is performing as expected\. In Amazon CloudWatch, you can create an alarm that sends a notification when the value of a certain metric is beyond what you consider an acceptable threshold\. 

Start by identifying the metric to monitor\. For example, you can configure an alarm to watch over the average CPU utilization of the EC2 instances in your Auto Scaling group\. The action can be a notification that is sent to you when the average CPU utilization of the group's instances breaches the threshold that you specified for the consecutive periods you specified\. For example, if the metric stays at or above 70 percent for 4 consecutive periods of 1 minute each\. 

For more information, see [Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) in the *Amazon CloudWatch User Guide*\.

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
   + For **Missing data treatment**, choose what you want the alarm to do if some data is missing\. For more information, see [Configuring How CloudWatch Alarms Treat Missing Data](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html#alarms-and-missing-data) in the *Amazon CloudWatch User Guide*\.

1. Choose **Next**\.

1. Under **Notification**, you can choose or create the Amazon SNS topic you want to use to receive notifications\. Otherwise, you can remove the notification now and add one later when you are ready\.

1. Choose **Next**\.

1. Enter a name and, optionally, a description for the alarm, and then choose **Next**\.

1. Choose **Create Alarm**\.