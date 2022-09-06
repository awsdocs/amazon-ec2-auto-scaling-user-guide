# Monitor CloudWatch metrics for your Auto Scaling groups and instances<a name="ec2-auto-scaling-cloudwatch-monitoring"></a>

*Metrics* are the fundamental concept in Amazon CloudWatch\. A metric represents a time\-ordered set of data points that are published to CloudWatch\. Think of a metric as a variable to monitor, and the data points as representing the values of that variable over time\. You can use these metrics to verify that your system is performing as expected\. 

Amazon EC2 Auto Scaling metrics that collect information about Auto Scaling groups are in the `AWS/AutoScaling` namespace\. Amazon EC2 instance metrics that collect CPU and other usage data from Auto Scaling instances are in the `AWS/EC2` namespace\. 

The Amazon EC2 Auto Scaling console displays a series of graphs for the group metrics and the aggregated instance metrics for the group\. Depending on your needs, you might prefer to access data for your Auto Scaling groups and instances from Amazon CloudWatch instead of the Amazon EC2 Auto Scaling console\.

For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.

**Topics**
+ [Available metrics and dimensions](#available-cloudwatch-metrics)
+ [Enable Auto Scaling group metrics \(console\)](#as-enable-group-metrics)
+ [Enable Auto Scaling group metrics \(AWS CLI\)](#as-enable-group-metrics-cli)
+ [Configure monitoring for Auto Scaling instances](enable-as-instance-metrics.md)
+ [View monitoring graphs in the Amazon EC2 Auto Scaling console](viewing-monitoring-graphs.md)

## Available metrics and dimensions<a name="available-cloudwatch-metrics"></a>

**Note**  
This section lists the different types of metrics in the `AWS/AutoScaling` namespace\.   
For information about the available metrics in the `AWS/EC2` namespace, see [List the available CloudWatch metrics for your instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html) in the *Amazon EC2 User Guide for Linux Instances*\. Or, see [Configure monitoring for Auto Scaling instances](enable-as-instance-metrics.md) to learn about how you can enable detailed monitoring for metrics in the `AWS/EC2` namespace or collect memory metrics from the EC2 instances in your Auto Scaling groups\.

Amazon EC2 Auto Scaling publishes the following metrics in the `AWS/AutoScaling` namespace\. 

Amazon EC2 Auto Scaling sends sampled data to CloudWatch every minute on a best\-effort basis\. In rare cases when CloudWatch experiences a service disruption, data isn't backfilled to fill gaps in group metric history\.

**Topics**
+ [Auto Scaling group metrics](#as-group-metrics)
+ [Dimensions for Auto Scaling group metrics](#as-group-metric-dimensions)
+ [Predictive scaling metrics and dimensions](#predictive-scaling-metrics)

### Auto Scaling group metrics<a name="as-group-metrics"></a>

When group metrics are enabled, Amazon EC2 Auto Scaling sends the following metrics to CloudWatch\. The metrics are available at one\-minute granularity at no additional charge, but you must enable them\. With these metrics, you get nearly continuous visibility into the history of your Auto Scaling group, such as changes in the size of the group over time\.


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

In addition to the metrics in the previous table, Amazon EC2 Auto Scaling also reports group metrics as an aggregate count of the number of capacity units that each instance represents\. If instance weighting is not applied, then the following metrics are populated, but are equal to the metrics that are defined in the previous table\. For more information about using weights, see [Configure instance weighting for Amazon EC2 Auto Scaling](ec2-auto-scaling-mixed-instances-groups-instance-weighting.md) and [Create an Auto Scaling group using attribute\-based instance type selection](create-asg-instance-type-requirements.md)\. 


| Metric | Description | 
| --- | --- | 
|  GroupInServiceCapacity  |  The number of capacity units that are running as part of the Auto Scaling group\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupPendingCapacity  |  The number of capacity units that are pending\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupStandbyCapacity  |  The number of capacity units that are in a `Standby` state\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupTerminatingCapacity  |  The number of capacity units that are in the process of terminating\.  **Reporting criteria**: Reported if metrics collection is enabled\.  | 
|  GroupTotalCapacity  |  The total number of capacity units in the Auto Scaling group\. **Reporting criteria**: Reported if metrics collection is enabled\.  | 

Amazon EC2 Auto Scaling also reports the following metrics for Auto Scaling groups that have a warm pool\. For more information, see [Warm pools for Amazon EC2 Auto Scaling](ec2-auto-scaling-warm-pools.md)\.


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

### Dimensions for Auto Scaling group metrics<a name="as-group-metric-dimensions"></a>

You can use the following dimensions to refine the metrics listed in the previous tables\.


| Dimension | Description | 
| --- | --- | 
|  AutoScalingGroupName  |  Filters on the name of an Auto Scaling group\.  | 

### Predictive scaling metrics and dimensions<a name="predictive-scaling-metrics"></a>

The `AWS/AutoScaling` namespace includes the following metrics for predictive scaling\. 

Metrics are available with a resolution of one hour\.

You can evaluate forecast accuracy by comparing forecasted values with actual values\. For more information about evaluating forecast accuracy using these metrics, see [Monitor predictive scaling metrics with CloudWatch](predictive-scaling-graphs.md#monitor-predictive-scaling-cloudwatch)\.


| Metric | Description | Dimensions | 
| --- | --- | --- | 
|  PredictiveScalingLoadForecast  |  The amount of load that's anticipated to be generated by your application\. The `Average`, `Minimum`, and `Maximum` statistics are useful, but the `Sum` statistic is not\.  **Reporting criteria**: Reported after the initial forecast is created\.  | AutoScalingGroupName, PolicyName, PairIndex  | 
| PredictiveScalingCapacityForecast |  The anticipated amount of capacity needed to meet application demand\. This is based on the load forecast and target utilization level at which you want to maintain your Auto Scaling instances\. The `Average`, `Minimum`, and `Maximum` statistics are useful, but the `Sum` statistic is not\. **Reporting criteria**: Reported after the initial forecast is created\.  | AutoScalingGroupName, PolicyName | 

**Note**  
The `PairIndex` dimension returns information associated with the index of the load\-scaling metric pair as assigned by Amazon EC2 Auto Scaling\. Currently, the only valid value is `0`\. 

## Enable Auto Scaling group metrics \(console\)<a name="as-enable-group-metrics"></a>

When you enable Auto Scaling group metrics, your Auto Scaling group sends sampled data to CloudWatch every minute\. There is no charge for enabling these metrics\.

**To enable group metrics**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom of the page\. 

1. On the **Monitoring** tab, select the **Auto Scaling group metrics collection**, **Enable** check box located at the top of the page under **Auto Scaling**\. 

**To disable group metrics**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select your Auto Scaling group\.

1. On the **Monitoring** tab, clear the **Auto Scaling group metrics collection**, **Enable** check box\. 

## Enable Auto Scaling group metrics \(AWS CLI\)<a name="as-enable-group-metrics-cli"></a>

**To enable group metrics**  
Enable one or more group metrics by using the [enable\-metrics\-collection](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/enable-metrics-collection.html) command\. For example, the following command enables the `GroupDesiredCapacity` metric\.

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
Use the [disable\-metrics\-collection](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/disable-metrics-collection.html) command\. For example, the following command disables all Auto Scaling group metrics\.

```
aws autoscaling disable-metrics-collection --auto-scaling-group-name my-asg
```