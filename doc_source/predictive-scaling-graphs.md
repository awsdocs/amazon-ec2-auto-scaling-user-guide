# Explore your data and forecast<a name="predictive-scaling-graphs"></a>

After the forecast is created, you can view graphs showing historical data from the last eight weeks and the forecast for the next two days\. The graphs become available shortly after the policy is created\.

**To view the forecast and its history using the Amazon EC2 Auto Scaling console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom of the page\. 

1. In the lower pane, choose the **Automatic scaling** tab\.

Each graph displays forecast values against actual values and a particular type of data\. The **Load** graph shows load forecast and actual values for the load metric you choose\. The **Capacity** graph shows the number of instances that are forecasted based on your target utilization and the actual number of instances launched\. Various colors show actual metric data points and past and future forecast values\. The orange line shows the actual metric data points\. The green line shows the forecast that was generated for the future forecast period\. The blue line shows the forecast for past periods\. 

![\[The graphs for a predictive scaling policy\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/asg-console-predictive-scaling-policy-graphs.png)

You can adjust the time range for past data by choosing your preferred value in the top right of the graph: **2 days**, **1 week**, **2 weeks**, **4 weeks**, **6 weeks**, or **8 weeks**\. Each point on the graph represents one hour of data\. When hovering over a data point, the tooltip shows the value for a particular point in time, in UTC\.

To enlarge the graph pane, choose the expand icon in the top right of the graph\. To revert back to the default view, choose the icon again\.

You can also use the AWS CLI command get\-predictive\-scaling\-forecast to get forecast data\. The data returned by this call can help you identify time periods when you might want to override the forecast\. For more information, see [Override forecast values using scheduled actions](predictive-scaling-overriding-forecast-capacity.md)\.

**Note**  
We recommend that you enable Auto Scaling group metrics\. If these metrics are not enabled, actual capacity data will be missing from the capacity forecast graph\. There is no cost for enabling these metrics\. For more information, see [Enable Auto Scaling group metrics \(console\)](ec2-auto-scaling-cloudwatch-monitoring.md#as-enable-group-metrics)\.

**Important**  
If the Auto Scaling group is new, allow 24 hours for Amazon EC2 Auto Scaling to create the first forecast\. 

## Monitor predictive scaling metrics with CloudWatch<a name="monitor-predictive-scaling-cloudwatch"></a>

Depending on your needs, you might prefer to access monitoring data for predictive scaling from Amazon CloudWatch instead of the Amazon EC2 Auto Scaling console\. After you create a predictive scaling policy, the policy collects data that is used to forecast your future load and capacity\. After this data is collected, it is automatically stored in CloudWatch at regular intervals\. Then, you can use CloudWatch to visualize how well the policy performs over time\. You can also create CloudWatch alarms to notify you when performance indicators change beyond the limits that you define in CloudWatch\.

**Topics**
+ [Visualize historical forecast data](#visualize-historical-forecast-data)
+ [Create accuracy metrics using metric math](#create-accuracy-metrics)

### Visualize historical forecast data<a name="visualize-historical-forecast-data"></a>

You can view the load and capacity forecast data for a predictive scaling policy in CloudWatch\. This can be useful when visualizing forecasts against other CloudWatch metrics in a single graph\. It can also help when viewing a broader time range so that you can see trends over time\. You can access up to 15 months of historical metrics to get a better perspective on how your policy is performing\.

For more information, see [Predictive scaling metrics and dimensions](ec2-auto-scaling-cloudwatch-monitoring.md#predictive-scaling-metrics)\.

**To view historical forecast data using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. In the navigation pane, choose **Metrics** and then **All metrics**\.

1. Choose the **Auto Scaling** metric namespace\.

1. Choose one of the following options to view either the load forecast or capacity forecast metrics: 
   + **Predictive Scaling Load Forecasts**
   + **Predictive Scaling Capacity Forecasts**

1. In the search field, enter the name of the predictive scaling policy or the name of the Auto Scaling group, and then press Enter to filter the results\. 

1. To graph a metric, select the check box next to the metric\. To change the name of the graph, choose the pencil icon\. To change the time range, select one of the predefined values or choose **custom**\. For more information, see [Graphing a metric](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/graph_a_metric.html) in the *Amazon CloudWatch User Guide*\.

1. To change the statistic, choose the **Graphed metrics** tab\. Choose the column heading or an individual value, and then choose a different statistic\. Although you can choose any statistic for each metric, not all statistics are useful for **PredictiveScalingLoadForecast** and **PredictiveScalingCapacityForecast** metrics\. For example, the **Average**, **Minimum**, and **Maximum** statistics are useful, but the **Sum** statistic is not\.

1. To add another metric to the graph, under **Browse**, choose **All**, find the specific metric, and then select the check box next to it\. You can add up to 10 metrics\.

   For example, to add the actual values for CPU utilization to the graph, choose the **EC2** namespace and then choose **By Auto Scaling Group**\. Then, select the check box for the **CPUUtilization** metric and the specific Auto Scaling group\. 

1. \(Optional\) To add the graph to a CloudWatch dashboard, choose **Actions**, **Add to dashboard**\.

### Create accuracy metrics using metric math<a name="create-accuracy-metrics"></a>

With metric math, you can query multiple CloudWatch metrics and use math expressions to create new time series based on these metrics\. You can visualize the resulting time series on the CloudWatch console and add them to dashboards\. For more information about metric math, see [Using metric math](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html) in the *Amazon CloudWatch User Guide*\.

Using metric math, you can graph the data that Amazon EC2 Auto Scaling generates for predictive scaling in different ways\. This helps you monitor policy performance over time, and helps you understand whether your combination of metrics can be improved\.

For example, you can use a metric math expression to monitor the [mean absolute percentage error](https://en.wikipedia.org/wiki/Mean_absolute_percentage_error) \(MAPE\)\. The MAPE metric helps monitor the difference between the forecasted values and the actual values observed during a given forecast window\. Changes in the value of MAPE can indicate whether the policy's performance is degrading over time as the nature of your application changes\. An increase in MAPE signals a wider gap between the forecasted values and the actual values\. 

**Example: Metric math expression**

To get started with this type of graph, you can create a metric math expression like the one shown in the following example\.

```
{
  "MetricDataQueries": [
    {
      "Expression": "TIME_SERIES(AVG(ABS(m1-m2)/m1))",
      "Id": "e1",
      "Period": 3600,
      "Label": "MeanAbsolutePercentageError",
      "ReturnData": true
    },
    {
      "Id": "m1",
      "Label": "ActualLoadValues",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/EC2",
          "MetricName": "CPUUtilization",
          "Dimensions": [
            {
              "Name": "AutoScalingGroupName",
              "Value": "my-asg"
            }
          ]
        },
        "Period": 3600,
        "Stat": "Sum"
      },
      "ReturnData": false
    },
    {
      "Id": "m2",
      "Label": "ForecastedLoadValues",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/AutoScaling",
          "MetricName": "PredictiveScalingLoadForecast",
          "Dimensions": [
            {
              "Name": "AutoScalingGroupName",
              "Value": "my-asg"
            },
            {
              "Name": "PolicyName",
              "Value": "my-predictive-scaling-policy"
            },
            {
              "Name": "PairIndex",
              "Value": "0"
            }
          ]
        },
        "Period": 3600,
        "Stat": "Average"
      },
      "ReturnData": false
    }
  ]
}
```

Instead of a single metric, there is an array of metric data query structures for `MetricDataQueries`\. Each item in `MetricDataQueries` gets a metric or performs a math expression\. The first item, `e1`, is the math expression\. The designated expression sets the `ReturnData` parameter to `true`, which ultimately produces a single time series\. For all other metrics, the `ReturnData` value is `false`\. 

In the example, the designated expression uses the actual and forecasted values as input and returns the new metric \(MAPE\)\. `m1` is the CloudWatch metric that contains the actual load values \(assuming CPU utilization is the load metric that was originally specified for the policy named `my-predictive-scaling-policy`\)\. `m2` is the CloudWatch metric that contains the forecasted load values\. The math syntax for the MAPE metric is as follows:

*Average of \(abs \(\(Actual \- Forecast\)/\(Actual\)\)\)*

#### Visualize your accuracy metrics and set alarms<a name="visualize-accuracy-metrics-set-alarms"></a>

To visualize the accuracy metric data, select the **Metrics** tab in the CloudWatch console\. You can graph the data from there\. For more information, see [Adding a math expression to a CloudWatch graph](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html#adding-metrics-expression-console) in the *Amazon CloudWatch User Guide*\.

You can also set an alarm on a metric that you're monitoring from the **Metrics** section\. While on the **Graphed metrics** tab, select the **Create alarm** icon under the **Actions** column\. The **Create alarm** icon is represented as a small bell\. For more information, see [Creating a CloudWatch alarm based on a metric math expression](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Create-alarm-on-metric-math-expression.html) in the *Amazon CloudWatch User Guide*\. For information about receiving alerts with Amazon SNS, see [Setting up Amazon SNS notifications](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/US_SetupSNS.html) in the *Amazon CloudWatch User Guide*\.

Alternatively, you can use [GetMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html) and [PutMetricAlarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricAlarm.html) to perform calculations using metric math and create alarms based on the output\.