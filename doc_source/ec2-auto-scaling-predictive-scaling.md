# Predictive scaling for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-predictive-scaling"></a>

Use predictive scaling to increase the number of EC2 instances in your Auto Scaling group in advance of daily and weekly patterns in traffic flows\. 

Predictive scaling is well suited for situations where you have:
+ Cyclical traffic, such as high use of resources during regular business hours and low use of resources during evenings and weekends
+ Recurring on\-and\-off workload patterns, such as batch processing, testing, or periodic data analysis
+ Applications that take a long time to initialize, causing a noticeable latency impact on application performance during scale\-out events

In general, if you have regular patterns of traffic increases and applications that take a long time to initialize, you should consider using predictive scaling\. Predictive scaling can help you scale faster by launching capacity in advance of forecasted load, compared to using only dynamic scaling, which is reactive in nature\. Predictive scaling can also potentially save you money on your EC2 bill by helping you avoid the need to overprovision capacity\.

For example, consider an application that has high usage during business hours and low usage overnight\. At the start of each business day, predictive scaling can add capacity before the first influx of traffic\. This helps your application maintain high availability and performance when going from a period of lower utilization to a period of higher utilization\. You don't have to wait for dynamic scaling to react to changing traffic\. You also don't have to spend time reviewing your application's load patterns and trying to schedule the right amount of capacity using scheduled scaling\. 

Use the AWS Management Console, the AWS CLI, or one of the SDKs to add a predictive scaling policy to any Auto Scaling group\.

**Topics**
+ [How predictive scaling works](#as-how-predictive-scaling-works)
+ [Best practices](#best-practices)
+ [Create a predictive scaling policy \(console\)](#predictive-scaling-policy-console)
+ [Create a predictive scaling policy \(AWS CLI\)](#predictive-scaling-policy-aws-cli)
+ [Limitations](#predictive-scaling-limitations)
+ [Supported Regions](#predictive-scaling-regions)
+ [Evaluate your predictive scaling policies](predictive-scaling-graphs.md)
+ [Override forecast values using scheduled actions](predictive-scaling-overriding-forecast-capacity.md)
+ [Advanced predictive scaling policy configurations using custom metrics](predictive-scaling-customized-metric-specification.md)

## How predictive scaling works<a name="as-how-predictive-scaling-works"></a>

Predictive scaling uses machine learning to predict capacity requirements based on historical data from CloudWatch\. The machine learning algorithm consumes the available historical data and calculates capacity that best fits the historical load pattern, and then continuously learns based on new data to make future forecasts more accurate\.

To use predictive scaling, you first create a scaling policy with a pair of metrics and a target utilization\. Forecast creation starts immediately after you create your policy if there is at least 24 hours of historical data\. Predictive scaling finds patterns in CloudWatch metric data from the previous 14 days to create an hourly forecast for the next 48 hours\. Forecast data is updated every six hours based on the most recent CloudWatch metric data\. 

You can configure predictive scaling in *forecast only* mode so that you can evaluate the forecast before predictive scaling starts actively scaling capacity\. You can then view the forecast and recent metric data from CloudWatch in graph form from the Amazon EC2 Auto Scaling console\. You can also access forecast data by using the AWS CLI or one of the SDKs\. 

When you are ready to start scaling with predictive scaling, switch the policy from *forecast only* mode to *forecast and scale* mode\. After you switch to *forecast and scale* mode, your Auto Scaling group starts scaling based on the forecast\. 

Using the forecast, Amazon EC2 Auto Scaling scales the number of instances at the beginning of each hour:
+ If actual capacity is less than the predicted capacity, Amazon EC2 Auto Scaling scales out your Auto Scaling group so that its desired capacity is equal to the predicted capacity\.
+ If actual capacity is greater than the predicted capacity, Amazon EC2 Auto Scaling doesn't scale in capacity\. 
+ The values that you set for the minimum and maximum capacity of the Auto Scaling group are adhered to if the predicted capacity is outside of this range\. 

## Best practices<a name="best-practices"></a>
+ Confirm whether predictive scaling is suitable for your workload\. A workload is a good fit for predictive scaling if it exhibits recurring load patterns that are specific to the day of the week or the time of day\. To check this, configure predictive scaling policies in *forecast only* mode and then refer to the recommendations in the console\. Amazon EC2 Auto Scaling provides recommendations based on observations about potential policy performance\. Evaluate the forecast and the recommendations before letting predictive scaling actively scale your application\.
+ Predictive scaling needs at least 24 hours of historical data to start forecasting\. However, forecasts are more effective if historical data spans two full weeks\. If you update your application by creating a new Auto Scaling group and deleting the old one, then your new Auto Scaling group needs 24 hours of historical load data before predictive scaling can start generating forecasts again\. You can use custom metrics to aggregate metrics across old and new Auto Scaling groups\. Otherwise, you might have to wait a few days for a more accurate forecast\. 
+ To get started, use the Amazon EC2 Auto Scaling console to create multiple predictive scaling policies in *forecast only* mode\. This tests the potential effects of different metrics and target values\. You can create multiple predictive scaling policies for each Auto Scaling group, but only one of the policies can be used for active scaling\.
+ When choosing a load metric, make sure that its data describes the full load on your application\. Also make sure that it's relevant to the performance aspect that you want to scale on\.
+ Use predictive scaling with dynamic scaling\. Dynamic scaling is used to automatically scale capacity in response to real\-time changes in resource utilization\. Using it with predictive scaling helps you follow the demand curve for your application closely, scaling in during periods of low traffic and scaling out when traffic is higher than expected\. When multiple scaling policies are active, each policy determines the desired capacity independently, and the desired capacity is set to the maximum of those\. For example, if 10 instances are required to stay at the target utilization in a target tracking scaling policy, and 8 instances are required to stay at the target utilization in a predictive scaling policy, then the group's desired capacity is set to 10\. We recommend using target tracking scaling policies for most customers who want to get started with dynamic scaling\. For more information, see [Target tracking scaling policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)\.

## Create a predictive scaling policy \(console\)<a name="predictive-scaling-policy-console"></a>

You can create, view, and delete your predictive scaling policies with the Amazon EC2 Auto Scaling console\. 

### Create a predictive scaling policy in the console \(predefined metrics\)<a name="create-a-predictive-scaling-policy-in-the-console"></a>

Use the following procedure to create a predictive scaling policy using predefined metrics \(CPU, network I/O, or Application Load Balancer request count per target\)\. The easiest way to create a predictive scaling policy is to use predefined metrics\. If you prefer to use custom metrics instead, see [Create a predictive scaling policy in the console \(custom metrics\)](#create-a-predictive-scaling-policy-in-the-console-custom-metrics)\.

If the Auto Scaling group is new, it must provide at least 24 hours of data before Amazon EC2 Auto Scaling can generate a forecast for it\. 

**To create a predictive scaling policy**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up at the bottom of the page\. 

1. On the **Automatic scaling** tab, in **Scaling policies**, choose **Create predictive scaling policy**\.

1. Enter a name for the policy\.

1. Turn on **Scale based on forecast** to give Amazon EC2 Auto Scaling permission to start scaling right away\.

   To keep the policy in *forecast only* mode, keep **Scale based on forecast** turned off\. 

1. For **Metrics**, choose your metrics from the list of options\. Options include **CPU**, **Network In**, **Network Out**, **Application Load Balancer request count**, and **Custom metric pair**\.

   If you chose **Application Load Balancer request count per target**, then choose a target group in **Target group**\. **Application Load Balancer request count per target** is only supported if you have attached an Application Load Balancer target group to your Auto Scaling group\. 

   If you chose **Custom metric pair**, choose individual metrics from the drop\-down lists for **Load metric** and **Scaling metric**\. 

1. For **Target utilization**, enter the target value that Amazon EC2 Auto Scaling should maintain\. Amazon EC2 Auto Scaling scales out your capacity until the average utilization is at the target utilization, or until it reaches the maximum number of instances you specified\.     
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-predictive-scaling.html)

1. \(Optional\) For **Pre\-launch instances**, choose how far in advance you want your instances launched before the forecast calls for the load to increase\. 

1. \(Optional\) For **Max capacity behavior**, choose whether to let Amazon EC2 Auto Scaling scale out higher than the group's maximum capacity when predicted capacity exceeds the defined maximum\. Turning on this setting lets scale out occur during periods when your traffic is forecasted to be at its highest\.

1. \(Optional\) For **Buffer maximum capacity above the forecasted capacity**, choose how much additional capacity to use when the predicted capacity is close to or exceeds the maximum capacity\. The value is specified as a percentage relative to the predicted capacity\. For example, if the buffer is 10, this means a 10 percent buffer\. Therefore, if the predicted capacity is 50 and the maximum capacity is 40, the effective maximum capacity is 55\. 

   If set to 0, Amazon EC2 Auto Scaling might scale capacity higher than the maximum capacity to equal but not exceed predicted capacity\.

1. Choose **Create predictive scaling policy**\.

### Create a predictive scaling policy in the console \(custom metrics\)<a name="create-a-predictive-scaling-policy-in-the-console-custom-metrics"></a>

Use the following procedure to create a predictive scaling policy using custom metrics\. Custom metrics can include other metrics provided by CloudWatch or metrics that you publish to CloudWatch\. To use CPU, network I/O, or Application Load Balancer request count per target, see [Create a predictive scaling policy in the console \(predefined metrics\)](#create-a-predictive-scaling-policy-in-the-console)\.

To create a predictive scaling policy using custom metrics, you must do the following:
+ You must provide the raw queries that let Amazon EC2 Auto Scaling interact with the metrics in CloudWatch\. For more information, see [Advanced predictive scaling policy configurations using custom metrics](predictive-scaling-customized-metric-specification.md)\. To be sure that Amazon EC2 Auto Scaling can extract the metric data from CloudWatch, confirm that each query is returning data points\. Confirm this by using the CloudWatch console or the CloudWatch [GetMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html) API operation\. 
**Note**  
We provide sample JSON payloads in the JSON editor in the Amazon EC2 Auto Scaling console\. These examples give you a reference for the key\-value pairs that are required to add other CloudWatch metrics provided by AWS or metrics that you previously published to CloudWatch\. You can use them as a starting point, then customize them for your needs\.
+ If you use any metric math, you must manually construct the JSON to fit your unique scenario\. For more information, see [Use metric math expressions](predictive-scaling-customized-metric-specification.md#using-math-expression-examples)\. Before using metric math in your policy, confirm that metric queries based on metric math expressions are valid and return a single time series\. Confirm this by using the CloudWatch console or the CloudWatch [GetMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html) API operation\.

If you make an error in a query by providing incorrect data, such as the wrong Auto Scaling group name, the forecast won't have any data\. For troubleshooting custom metric issues, see [Considerations and troubleshooting](predictive-scaling-customized-metric-specification.md#custom-metrics-troubleshooting)\.

**To create a predictive scaling policy**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up at the bottom of the page\. 

1. On the **Automatic scaling** tab, in **Scaling policies**, choose **Create predictive scaling policy**\.

1. Enter a name for the policy\.

1. Turn on **Scale based on forecast** to give Amazon EC2 Auto Scaling permission to start scaling right away\.

   To keep the policy in *forecast only* mode, keep **Scale based on forecast** turned off\. 

1. For **Metrics**, choose **Custom metric pair**\.

   1. For **Load metric**, choose **Custom CloudWatch metric** to use a custom metric\. Construct the JSON payload that contains the load metric definition for the policy and paste it into the JSON editor box, replacing what is already in the box\.

   1. For **Scaling metric**, choose **Custom CloudWatch metric** to use a custom metric\. Construct the JSON payload that contains the scaling metric definition for the policy and paste it into the JSON editor box, replacing what is already in the box\. 

   1. \(Optional\) To add a custom capacity metric, select the check box for **Add custom capacity metric**\. Construct the JSON payload that contains the capacity metric definition for the policy and paste it into the JSON editor box, replacing what is already in the box\.

      You only need to enable this option to create a new time series for capacity if your capacity metric data spans multiple Auto Scaling groups\. In this case, you must use metric math to aggregate the data into a single time series\.

1. For **Target utilization**, enter the target value that Amazon EC2 Auto Scaling should maintain\. Amazon EC2 Auto Scaling scales out your capacity until the average utilization is at the target utilization, or until it reaches the maximum number of instances you specified\. 

1. \(Optional\) For **Pre\-launch instances**, choose how far in advance you want your instances launched before the forecast calls for the load to increase\. 

1. \(Optional\) For **Max capacity behavior**, choose whether to let Amazon EC2 Auto Scaling scale out higher than the group's maximum capacity when predicted capacity exceeds the defined maximum\. Turning on this setting lets scale out occur during periods when your traffic is forecasted to be at its highest\.

1. \(Optional\) For **Buffer maximum capacity above the forecasted capacity**, choose how much additional capacity to use when the predicted capacity is close to or exceeds the maximum capacity\. The value is specified as a percentage relative to the predicted capacity\. For example, if the buffer is 10, this means a 10 percent buffer\. Therefore, if the predicted capacity is 50 and the maximum capacity is 40, the effective maximum capacity is 55\. 

   If set to 0, Amazon EC2 Auto Scaling might scale capacity higher than the maximum capacity to equal but not exceed predicted capacity\.

1. Choose **Create predictive scaling policy**\.

## Create a predictive scaling policy \(AWS CLI\)<a name="predictive-scaling-policy-aws-cli"></a>

Use the AWS CLI as follows to configure predictive scaling policies for your Auto Scaling group\. For more information about the CloudWatch metrics you can specify for a predictive scaling policy, see [PredictiveScalingMetricSpecification](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PredictiveScalingMetricSpecification.html) in the *Amazon EC2 Auto Scaling API Reference*\.

### Example 1: A predictive scaling policy that creates forecasts but doesn't scale<a name="predictive-scaling-configuration-ex1"></a>

The following example policy shows a complete policy configuration that uses CPU utilization metrics for predictive scaling with a target utilization of `40`\. `ForecastOnly` mode is used by default, unless you explicitly specify which mode to use\. Save this configuration in a file named `config.json`\.

```
{
    "MetricSpecifications": [
        {
            "TargetValue": 40,
            "PredefinedMetricPairSpecification": {
                "PredefinedMetricType": "ASGCPUUtilization"
            }
        }
    ]
}
```

To create the policy from the command line, run the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command with the configuration file specified, as demonstrated in the following example\.

```
aws autoscaling put-scaling-policy --policy-name cpu40-predictive-scaling-policy \
  --auto-scaling-group-name my-asg --policy-type PredictiveScaling \
  --predictive-scaling-configuration file://config.json
```

If successful, this command returns the policy's Amazon Resource Name \(ARN\)\.

```
{
  "PolicyARN": "arn:aws:autoscaling:region:account-id:scalingPolicy:2f4f5048-d8a8-4d14-b13a-d1905620f345:autoScalingGroupName/my-asg:policyName/cpu40-predictive-scaling-policy",
  "Alarms": []
}
```

### Example 2: A predictive scaling policy that forecasts and scales<a name="predictive-scaling-configuration-ex2"></a>

For a policy that allows Amazon EC2 Auto Scaling to forecast and scale, add the property `Mode` with a value of `ForecastAndScale`\. The following example shows a policy configuration that uses Application Load Balancer request count metrics\. The target utilization is `1000`, and predictive scaling is set to `ForecastAndScale` mode\.

```
{
    "MetricSpecifications": [
        {
            "TargetValue": 1000,
            "PredefinedMetricPairSpecification": {
                "PredefinedMetricType": "ALBRequestCount",
                "ResourceLabel": "app/my-alb/778d41231b141a0f/targetgroup/my-alb-target-group/943f017f100becff"
            }
        }
    ],
    "Mode": "ForecastAndScale"
}
```

To create this policy, run the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command with the configuration file specified, as demonstrated in the following example\.

```
aws autoscaling put-scaling-policy --policy-name alb1000-predictive-scaling-policy \
  --auto-scaling-group-name my-asg --policy-type PredictiveScaling \
  --predictive-scaling-configuration file://config.json
```

If successful, this command returns the policy's Amazon Resource Name \(ARN\)\.

```
{
  "PolicyARN": "arn:aws:autoscaling:region:account-id:scalingPolicy:19556d63-7914-4997-8c81-d27ca5241386:autoScalingGroupName/my-asg:policyName/alb1000-predictive-scaling-policy",
  "Alarms": []
}
```

### Example 3: A predictive scaling policy that can scale higher than maximum capacity<a name="predictive-scaling-configuration-ex3"></a>

The following example shows how to create a policy that can scale higher than the group's maximum size limit when you need it to handle a higher than normal load\. By default, Amazon EC2 Auto Scaling doesn't scale your EC2 capacity higher than your defined maximum capacity\. However, it might be helpful to let it scale higher with slightly more capacity to avoid performance or availability issues\.

To provide room for Amazon EC2 Auto Scaling to provision additional capacity when the capacity is predicted to be at or very close to your group's maximum size, specify the `MaxCapacityBreachBehavior` and `MaxCapacityBuffer` properties, as shown in the following example\. You must specify `MaxCapacityBreachBehavior` with a value of `IncreaseMaxCapacity`\. The maximum number of instances that your group can have depends on the value of `MaxCapacityBuffer`\. 

```
{
    "MetricSpecifications": [
        {
            "TargetValue": 70,
            "PredefinedMetricPairSpecification": {
                "PredefinedMetricType": "ASGCPUUtilization"
            }
        }
    ],
    "MaxCapacityBreachBehavior": "IncreaseMaxCapacity",
    "MaxCapacityBuffer": 10
}
```

In this example, the policy is configured to use a 10 percent buffer \(`"MaxCapacityBuffer": 10`\), so if the predicted capacity is 50 and the maximum capacity is 40, then the effective maximum capacity is 55\. A policy that can scale capacity higher than the maximum capacity to equal but not exceed predicted capacity would have a buffer of 0 \(`"MaxCapacityBuffer": 0`\)\. 

To create this policy, run the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command with the configuration file specified, as demonstrated in the following example\.

```
aws autoscaling put-scaling-policy --policy-name cpu70-predictive-scaling-policy \
  --auto-scaling-group-name my-asg --policy-type PredictiveScaling \
  --predictive-scaling-configuration file://config.json
```

If successful, this command returns the policy's Amazon Resource Name \(ARN\)\.

```
{
  "PolicyARN": "arn:aws:autoscaling:region:account-id:scalingPolicy:d02ef525-8651-4314-bf14-888331ebd04f:autoScalingGroupName/my-asg:policyName/cpu70-predictive-scaling-policy",
  "Alarms": []
}
```

## Limitations<a name="predictive-scaling-limitations"></a>
+ Predictive scaling requires 24 hours of metric history before it can generate forecasts\.
+ A core assumption of predictive scaling is that the Auto Scaling group is homogenous and all instances are of equal capacity\. If this isn’t true for your group, forecasted capacity can be inaccurate\. Therefore, use caution when creating predictive scaling policies for [mixed instances groups](ec2-auto-scaling-mixed-instances-groups.md), because instances of different types can be provisioned that are of unequal capacity\. Following are some examples where the forecasted capacity will be inaccurate:
  + Your predictive scaling policy is based on CPU utilization, but the number of vCPUs on each Auto Scaling instance varies between instance types\.
  + Your predictive scaling policy is based on network in or network out, but the network bandwidth throughput for each Auto Scaling instance varies between instance types\. For example, the M5 and M5n instance types are similar, but the M5n instance type delivers significantly higher network throughput\.

## Supported Regions<a name="predictive-scaling-regions"></a>

Amazon EC2 Auto Scaling supports predictive scaling policies in the following AWS Regions: US East \(N\. Virginia\), US East \(Ohio\), US West \(Oregon\), US West \(N\. California\), Africa \(Cape Town\), Canada \(Central\), EU \(Frankfurt\), EU \(Ireland\), EU \(London\), EU \(Milan\), EU \(Paris\), EU \(Stockholm\), Asia Pacific \(Hong Kong\), Asia Pacific \(Jakarta\), Asia Pacific \(Mumbai\), Asia Pacific \(Osaka\), Asia Pacific \(Tokyo\), Asia Pacific \(Singapore\), Asia Pacific \(Seoul\), Asia Pacific \(Sydney\), Middle East \(Bahrain\), South America \(Sao Paulo\), China \(Beijing\), China \(Ningxia\), and AWS GovCloud \(US\-West\)\. 