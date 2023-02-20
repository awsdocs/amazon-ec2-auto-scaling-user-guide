# Evaluate your predictive scaling policies<a name="predictive-scaling-graphs"></a>

Before you use a predictive scaling policy to scale your Auto Scaling group, review the recommendations and other data for your policy in the Amazon EC2 Auto Scaling console\. This is important because you don't want a predictive scaling policy to scale your actual capacity until you know that its predictions are accurate\.

If the Auto Scaling group is new, give Amazon EC2 Auto Scaling 24 hours to create the first forecast\.

When Amazon EC2 Auto Scaling creates a forecast, it uses historical data\. If your Auto Scaling group doesn't have much recent historical data yet, Amazon EC2 Auto Scaling might temporarily backfill the forecast with aggregates created from the currently available historical aggregates\. Forecasts are backfilled for up to two weeks before a policy's creation date\.

**Topics**
+ [View your recommendations](#view-predictive-scaling-recommendations)
+ [Review monitoring graphs](#review-predictive-scaling-monitoring-graphs)
+ [Monitor metrics with CloudWatch](#monitor-predictive-scaling-cloudwatch)

## View your predictive scaling recommendations<a name="view-predictive-scaling-recommendations"></a>

For effective analysis, Amazon EC2 Auto Scaling should have at least two predictive scaling policies to compare\. \(However, you can still review the findings for a single policy\.\) When you create multiple policies, you can evaluate a policy that uses one metric against a policy that uses a different metric\. You can also evaluate the impact of different target value and metric combinations\. After the predictive scaling policies are created, Amazon EC2 Auto Scaling immediately starts evaluating which policy would do a better job of scaling your group\.

**To view your recommendations in the Amazon EC2 Auto Scaling console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom of the page\.

1. On the **Auto scaling** tab, under **Predictive scaling policies**, you can view details about a policy along with our recommendation\. The recommendation tells you whether the predictive scaling policy does a better job than not using it\. 

   If you're unsure whether a predictive scaling policy is appropriate for your group, review the **Availability impact** and **Cost impact** columns to choose the right policy\. The information for each column tells you what the impact of the policy is\. 
   + **Availability impact**: Describes whether the policy would avoid negative impact to availability by provisioning enough instances to handle the workload, compared to not using the policy\.
   + **Cost impact**: Describes whether the policy would avoid negative impact on your costs by not over\-provisioning instances, compared to not using the policy\. By over\-provisioning too much, your instances are underutilized or idle, which only adds to the cost impact\.

   If you have multiple policies, then a **Best prediction** tag displays next to the name of the policy that gives the most availability benefits at lower cost\. More weight is given to availability impact\. 

1. \(Optional\) To select the desired time period for recommendation results, choose your preferred value from the **Evaluation period** dropdown: **2 days**, **1 week**, **2 weeks**, **4 weeks**, **6 weeks**, or **8 weeks**\. By default, the evaluation period is the last two weeks\. A longer evaluation period provides more data points to the recommendation results\. However, adding more data points might not improve the results if your load patterns have changed, such as after a period of exceptional demand\. In this case, you can get a more focused recommendation by looking at more recent data\.

**Note**  
Recommendations are generated only for policies that are in **Forecast only** mode\. The recommendations feature works better when a policy is in the **Forecast only** mode throughout the evaluation period\. If you start a policy in **Forecast and scale** mode and switch it to **Forecast only** mode later, the findings for that policy are likely to be biased\. This is because the policy has already contributed toward the actual capacity\.

## Review predictive scaling monitoring graphs<a name="review-predictive-scaling-monitoring-graphs"></a>

In the Amazon EC2 Auto Scaling console, you can review the forecast of the previous days, weeks, or months to visualize how well the policy performs over time\. You can also use this information to evaluate the accuracy of predictions when deciding whether to let a policy scale your actual capacity\.

**To review predictive scaling monitoring graphs in the Amazon EC2 Auto Scaling console**

1. Choose a policy from the **Predictive scaling policies** list\. 

1. In the **Monitoring** section, you can view your policy's past and future forecasts for load and capacity against actual values\. The **Load** graph shows load forecast and actual values for the load metric that you chose\. The **Capacity** graph shows the number of instances predicted by the policy\. It also includes the actual number of instances launched\. The vertical line separates historical values from future forecasts\. These graphs become available shortly after the policy is created\. 

1. \(Optional\) To change the amount of historical data shown in the chart, choose your preferred value from the **Evaluation period** dropdown at the top of the page\. The evaluation period does not transform the data on this page in any way\. It only changes the amount of historical data shown\.

The following image shows the **Load** and **Capacity** graphs when forecasts have been applied multiple times\. Predictive scaling forecasts load based on your historical load data\. The load your application generates is represented as the sum of the CPU utilization, network in/out, received requests, or custom metric for each instance in the Auto Scaling group\. Predictive scaling calculates future capacity needs based on the load forecast and the target utilization that you want to achieve for the scaling metric\.

![\[Predictive scaling graphs\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/predictive-scaling-graphs.png)![\[Predictive scaling graphs\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[Predictive scaling graphs\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

**Compare data in the **Load** graph**  
Each horizontal line represents a different set of data points reported in one\-hour intervals:

1. **Actual observed load** uses the SUM statistic for your chosen load metric to show the total hourly load in the past\.

1. **Load predicted by the policy** shows the hourly load prediction\. This prediction is based on the previous two weeks of actual load observations\.

**Compare data in the **Capacity** graph**  
Each horizontal line represents a different set of data points reported in one\-hour intervals:

1. **Actual observed capacity** shows your Auto Scaling group's actual capacity in the past, which depends on your other scaling policies and minimum group size in effect for the selected time period\.

1. **Capacity predicted by the policy** shows the baseline capacity that you can expect to have at the beginning of each hour when the policy is in **Forecast and scale** mode\.

1. **Inferred required capacity** shows the ideal capacity to maintain the scaling metric at the target value you chose\.

1. **Minimum capacity** shows the minimum capacity of the Auto Scaling group\.

1. **Maximum capacity** shows the maximum capacity of the Auto Scaling group\.

For the purpose of calculating the inferred required capacity, we begin by assuming that each instance is equally utilized at a specified target value\. In practice, instances are not equally utilized\. By assuming that utilization is uniformly spread between instances, however, we can make a likelihood estimate of the amount of capacity that is needed\. The capacity requirement is then calculated to be inversely proportional to the scaling metric that you used for your predictive scaling policy\. In other words, as capacity increases, the scaling metric decreases at the same rate\. For example, if capacity doubles, the scaling metric must decrease by half\. 

The formula for the inferred required capacity:

 `sum of (actualCapacityUnits*scalingMetricValue)/(targetUtilization)`

For example, we take the `actualCapacityUnits` \(`10`\) and the `scalingMetricValue` \(`30`\) for a given hour\. We then take the `targetUtilization` that you specified in your predictive scaling policy \(`60`\) and calculate the inferred required capacity for the same hour\. This returns a value of `5`\. This means that five is the inferred amount of capacity required to maintain capacity in direct inverse proportion to the target value of the scaling metric\.

**Note**  
Various levers are available for you to adjust and improve the cost savings and availability of your application\.  
You use predictive scaling for the baseline capacity and dynamic scaling to handle additional capacity\. Dynamic scaling works independently from predictive scaling, scaling in and out based on current utilization\. First, Amazon EC2 Auto Scaling calculates the recommended number of instances for each dynamic scaling policy\. Then, it scales based on the policy that provides the largest number of instances\.
To allow scale in to occur when the load decreases, your Auto Scaling group should always have at least one dynamic scaling policy with the scale\-in portion enabled\.
You can improve scaling performance by making sure that your minimum and maximum capacity are not too restrictive\. A policy with a recommended number of instances that does not fall within the minimum and maximum capacity range will be prevented from scaling in and out\.

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