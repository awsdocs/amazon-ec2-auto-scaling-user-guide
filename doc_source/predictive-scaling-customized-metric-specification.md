# Advanced predictive scaling policy configurations using custom metrics<a name="predictive-scaling-customized-metric-specification"></a>

In a predictive scaling policy, you can use predefined or custom metrics\. The custom metrics you specify can include other metrics available in CloudWatch and new metrics that you publish to CloudWatch\. Custom metrics are useful when the predefined metrics \(CPU, network I/O, and Application Load Balancer request count\) do not sufficiently describe your application load\.

Metric math expressions can also be used to aggregate the metric data received from CloudWatch before using it for predictive scaling\. When you combine values in your data, for example, by calculating new sums or averages, it's called *aggregating*\. The resulting data is called an *aggregate*\. 

On the graphs screen in the Amazon EC2 Auto Scaling console, you can view **Load** and **Capacity** graphs with your metric data in the same way that you do when specifying predefined metrics\. However, you currently can't use the Amazon EC2 Auto Scaling console to create or update a predictive scaling policy that specifies custom metrics\. To do so, you must use the AWS CLI or an AWS SDK\. 

**Topics**
+ [Best practices](#custom-metrics-best-practices)
+ [Prerequisites](#custom-metrics-prerequisites)
+ [Example predictive scaling policy with a custom scaling metric and a custom load metric](#custom-metrics-ex1)
+ [Using metric math expressions](#using-math-expression-examples)
+ [Considerations and troubleshooting](#custom-metrics-troubleshooting)
+ [Limitations](#custom-metrics-limitations)

## Best practices<a name="custom-metrics-best-practices"></a>

The following best practices can help you use custom metrics more effectively:
+ For the load metric specification, the most useful metric is a metric that represents the load on an Auto Scaling group as a whole, regardless of the group's capacity\.
+ For the scaling metric specification, the most useful metric to scale by is an average throughput or utilization per instance metric\.
+ The scaling metric must be inversely proportional to capacity\. That is, if the number of instances in the Auto Scaling group increases, the scaling metric should decrease by roughly the same proportion\. To ensure that predictive scaling behaves as expected, the load metric and the scaling metric must also correlate strongly with each other\. 
+ The target utilization must match the type of scaling metric\. For a policy configuration that uses CPU utilization, this is a target percentage\. For a policy configuration that uses throughput, such as the number of requests or messages, this is the target number of requests or messages per instance during any one\-minute interval\.
+ If these recommendations are not followed, the forecasted future values of the time series will probably be incorrect\. To validate that the data is correct, you can view the forecasted values in the Amazon EC2 Auto Scaling console\. Alternatively, after you create your predictive scaling policy, inspect the `LoadForecast` and `CapacityForecast` objects returned by a call to the [GetPredictiveScalingForecast](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_GetPredictiveScalingForecast.html) API\.
+ We strongly recommend that you configure predictive scaling in forecast only mode so that you can evaluate the forecast before predictive scaling starts actively scaling capacity\.

## Prerequisites<a name="custom-metrics-prerequisites"></a>

To specify custom metrics in your policy, you must have `cloudwatch:GetMetricData` permissions\.

To specify your own metrics instead of the metrics that AWS provides, you must first publish the metrics to CloudWatch\. For more information, see [Publishing custom metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/publishingMetrics.html) in the *Amazon CloudWatch User Guide*\. 

**Note**  
When you publish your own metrics, make sure to publish the data points at a minimum frequency of five minutes\. Amazon EC2 Auto Scaling retrieves the data points from CloudWatch based on the length of the period that it needs\. For example, the load metric specification uses hourly metrics to measure the load on your application\. CloudWatch uses your published metric data to provide a single data value for any one\-hour period by aggregating all data points with timestamps that fall within each one\-hour period\. 

## Example predictive scaling policy with a custom scaling metric and a custom load metric<a name="custom-metrics-ex1"></a>

The following example policy shows a complete policy configuration that specifies a custom scaling metric, a custom load metric, and a target utilization of `50`\. Save this configuration in a file named `config.json`\.

```
{
  "MetricSpecifications": [
    {
      "TargetValue": 50,
      "CustomizedScalingMetricSpecification": {
        "MetricDataQueries": [
          {
            "MetricStat": {
              "Metric": {
                "MetricName": "MyUtilizationMetric",
                "Namespace": "MyNameSpace",
                "Dimensions": [
                  {
                    "Name": "MyOptionalMetricDimensionName",
                    "Value": "MyOptionalMetricDimensionValue"
                  }
                ]
              },
              "Stat": "Average"
            }
          }
        ]
      },
      "CustomizedLoadMetricSpecification": {
        "MetricDataQueries": [
          {
            "MetricStat": {
              "Metric": {
                "MetricName": "MyLoadMetric",
                "Namespace": "MyNameSpace",
                "Dimensions": [
                  {
                    "Name": "MyOptionalMetricDimensionName",
                    "Value": "MyOptionalMetricDimensionValue"
                  }
                ]
              },
              "Stat": "Sum"
            }
          }
        ]
      }
    }
  ]
}
```

For information about the Amazon EC2 Auto Scaling parameters and values for a metric data query, see [MetricDataQuery](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_MetricDataQuery.html) in the *Amazon EC2 Auto Scaling API Reference*\.

**Note**  
Following are some additional resources that can help:   
For information about the available metrics for AWS services, see [AWS services that publish CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html) in the *Amazon CloudWatch User Guide*\.
To get the exact metric name, namespace, and dimensions \(if applicable\) for a CloudWatch metric with the AWS CLI, see [list\-metrics](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/list-metrics.html)\. 

To create this policy, run the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command with the configuration file specified, as demonstrated in the following example\.

```
aws autoscaling put-scaling-policy --policy-name my-predictive-scaling-policy \
  --auto-scaling-group-name my-asg --policy-type PredictiveScaling \
  --predictive-scaling-configuration file://config.json
```

If successful, this command returns the policy's Amazon Resource Name \(ARN\)\.

```
{
  "PolicyARN": "arn:aws:autoscaling:region:account-id:scalingPolicy:2f4f5048-d8a8-4d14-b13a-d1905620f345:autoScalingGroupName/my-asg:policyName/my-predictive-scaling-policy",
  "Alarms": []
}
```

## Using metric math expressions<a name="using-math-expression-examples"></a>

The following AWS CLI examples might be relevant to your scenario when creating a predictive scaling policy that uses metric math expressions\. You can use these example predictive scaling policy configurations as a starting point, then customize them for your needs\.

**Topics**
+ [Understanding metric math](#custom-metrics-metric-math)
+ [Example predictive scaling policy that combines metrics using metric math](#custom-metrics-ex2)
+ [Example predictive scaling policy to use in a blue/green deployment scenario](#custom-metrics-ex3)

### Understanding metric math<a name="custom-metrics-metric-math"></a>

If all you want to do is aggregate existing metric data, CloudWatch metric math saves you the effort and cost of publishing another metric to CloudWatch\. You can use any metric that AWS provides, and you can also use metrics that you define as part of your applications\. For example, you might want to calculate the Amazon SQS queue backlog per instance\. You can do this by taking the approximate number of messages available for retrieval from the queue and dividing that number by the Auto Scaling group's running capacity\.

For more information, see [Using metric math](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html) in the *Amazon CloudWatch User Guide*\. 

If you choose to use a metric math expression in your predictive scaling policy, consider the following points:
+ Metric math operations use the data points of the unique combination of metric name, namespace, and dimension keys/value pairs of metrics\. 
+ You can use any arithmetic operator \(\+ \- \* / ^\), statistical function \(such as AVG or SUM\), or other function that CloudWatch supports\. 
+ You can use both metrics and the results of other math expressions in the formulas of the math expression\. 
+ Your metric math expressions can be made up of different aggregations\. However, it's a best practice for the final aggregation result to use `Average` for the scaling metric and `Sum` for the load metric\.
+ Any expressions used in a metric specification must eventually return a single time series\.

To use metric math, do the following:
+ Choose one or more CloudWatch metrics\. Then, create the expression\. For more information, see [Using metric math](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html) in the *Amazon CloudWatch User Guide*\. 
+ Verify that the metric math expression is valid by using the CloudWatch console or the CloudWatch [GetMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html) API\.

### Example predictive scaling policy that combines metrics using metric math<a name="custom-metrics-ex2"></a>

Sometimes, instead of specifying the metric directly, you might need to first process its data in some way\. For example, you might have an application that pulls work from an Amazon SQS queue, and you might want to use the number of items in the queue as criteria for predictive scaling\. The number of messages in the queue does not solely define the number of instances that you need\. Therefore, more work is needed to create a metric that can be used to calculate the backlog per instance\. For more information, see [Scaling based on Amazon SQS](as-using-sqs-queue.md)\.

The following is an example predictive scaling policy for this scenario\. It specifies scaling and load metrics that are based on the Amazon SQS `ApproximateNumberOfMessagesVisible` metric, which is the number of messages available for retrieval from the queue\. It also uses the Amazon EC2 Auto Scaling `GroupInServiceInstances` metric and a math expression to calculate the backlog per instance for the scaling metric\.

```
aws autoscaling put-scaling-policy --policy-name my-sqs-custom-metrics-policy \
  --auto-scaling-group-name my-asg --policy-type PredictiveScaling \
  --predictive-scaling-configuration file://config.json
{
  "MetricSpecifications": [
    {
      "TargetValue": 100,
      "CustomizedScalingMetricSpecification": {
        "MetricDataQueries": [
          {
            "Label": "Get the queue size (the number of messages waiting to be processed)",
            "Id": "queue_size",
            "MetricStat": {
              "Metric": {
                "MetricName": "ApproximateNumberOfMessagesVisible",
                "Namespace": "AWS/SQS",
                "Dimensions": [
                  {
                    "Name": "QueueName",
                    "Value": "my-queue"
                  }
                ]
              },
              "Stat": "Sum"
            },
            "ReturnData": false
          },
          {
            "Label": "Get the group size (the number of running instances)",
            "Id": "running_capacity",
            "MetricStat": {
              "Metric": {
                "MetricName": "GroupInServiceInstances",
                "Namespace": "AWS/AutoScaling",
                "Dimensions": [
                  {
                    "Name": "AutoScalingGroupName",
                    "Value": "my-asg"
                  }
                ]
              },
              "Stat": "Sum"
            },
            "ReturnData": false
          },
          {
            "Label": "Calculate the backlog per instance",
            "Id": "scaling_metric",
            "Expression": "queue_size / running_capacity",
            "ReturnData": true
          }
        ]
      },
      "CustomizedLoadMetricSpecification": {
        "MetricDataQueries": [
          {
            "Id": "load_metric",
            "MetricStat": {
              "Metric": {
                "MetricName": "ApproximateNumberOfMessagesVisible",
                "Namespace": "AWS/SQS",
                "Dimensions": [
                  {
                    "Name": "QueueName",
                    "Value": "my-queue"
                  }
                ],
                "Stat": "Sum"
              }
            },
            "ReturnData": true
          }
        ]
      }
    }
  ]
}
```

The example returns the policy's ARN\.

```
{
  "PolicyARN": "arn:aws:autoscaling:region:account-id:scalingPolicy:2f4f5048-d8a8-4d14-b13a-d1905620f345:autoScalingGroupName/my-asg:policyName/my-sqs-custom-metrics-policy",
  "Alarms": []
}
```

### Example predictive scaling policy to use in a blue/green deployment scenario<a name="custom-metrics-ex3"></a>

A search expression provides an advanced option in which you can query for a metric from multiple Auto Scaling groups and perform math expressions on them\. This is especially useful for blue/green deployments\. 

**Note**  
A *blue/green deployment* is a deployment method in which you create two separate but identical Auto Scaling groups\. Only one of the groups receives production traffic\. User traffic is initially directed to the earlier \("blue"\) Auto Scaling group, while a new group \("green"\) is used for testing and evaluation of a new version of an application or service\. User traffic is shifted to the green Auto Scaling group after a new deployment is tested and accepted\. You can then delete the blue group after the deployment is successful\.

When new Auto Scaling groups get created as part of a blue/green deployment, the metric history of each group can be automatically included in the predictive scaling policy without you having to change its metric specifications\. For more information, see [Using EC2 Auto Scaling predictive scaling policies with Blue/Green deployments](http://aws.amazon.com/blogs/compute/retaining-metrics-across-blue-green-deployment-for-predictive-scaling/) on the AWS Compute Blog\.

The following example policy shows how this can be done\. In this example, the policy uses the `CPUUtilization` metric emitted by Amazon EC2\. It uses the Amazon EC2 Auto Scaling `GroupInServiceInstances` metric and a math expression to calculate the value of the scaling metric per instance\. It also specifies a capacity metric specification to get the `GroupInServiceInstances` metric\.

The search expression finds the `CPUUtilization` of instances in multiple Auto Scaling groups based on the specified search criteria\. If you later create a new Auto Scaling group that matches the same search criteria, the `CPUUtilization` of the instances in the new Auto Scaling group is automatically included\.

```
aws autoscaling put-scaling-policy --policy-name my-blue-green-predictive-scaling-policy \
  --auto-scaling-group-name my-asg --policy-type PredictiveScaling \
  --predictive-scaling-configuration file://config.json
{
  "MetricSpecifications": [
    {
      "TargetValue": 25,
      "CustomizedScalingMetricSpecification": {
        "MetricDataQueries": [
          {
            "Id": "load_sum",
            "Expression": "SUM(SEARCH('{AWS/EC2,AutoScalingGroupName} MetricName=\"CPUUtilization\" ASG-myapp', 'Sum', 300))",
            "ReturnData": false
          },
          {
            "Id": "capacity_sum",
            "Expression": "SUM(SEARCH('{AWS/AutoScaling,AutoScalingGroupName} MetricName=\"GroupInServiceInstances\" ASG-myapp', 'Average', 300))",
            "ReturnData": false
          },
          {
            "Id": "weighted_average",
            "Expression": "load_sum / capacity_sum",
            "ReturnData": true
          }
        ]
      },
      "CustomizedLoadMetricSpecification": {
        "MetricDataQueries": [
          {
            "Id": "load_sum",
            "Expression": "SUM(SEARCH('{AWS/EC2,AutoScalingGroupName} MetricName=\"CPUUtilization\" ASG-myapp', 'Sum', 3600))"
          }
        ]
      },
      "CustomizedCapacityMetricSpecification": {
        "MetricDataQueries": [
          {
            "Id": "capacity_sum",
            "Expression": "SUM(SEARCH('{AWS/AutoScaling,AutoScalingGroupName} MetricName=\"GroupInServiceInstances\" ASG-myapp', 'Average', 300))"
          }
        ]
      }
    }
  ]
}
```

The example returns the policy's ARN\.

```
{
  "PolicyARN": "arn:aws:autoscaling:region:account-id:scalingPolicy:2f4f5048-d8a8-4d14-b13a-d1905620f345:autoScalingGroupName/my-asg:policyName/my-blue-green-predictive-scaling-policy",
  "Alarms": []
}
```

## Considerations and troubleshooting<a name="custom-metrics-troubleshooting"></a>

If an issue occurs while using custom metrics, we recommend that you do the following:
+ If an error message is provided, read the message and resolve the issue it reports, if possible\. 
+ If an issue occurs when you are trying to use a search expression in a blue/green deployment scenario, first make sure that you understand how to create a search expression that looks for a partial match instead of an exact match\. Also, check that your query finds only the Auto Scaling groups that are running the specific application\. For more information about the search expression syntax, see [CloudWatch search expression syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/search-expression-syntax.html) in the *Amazon CloudWatch User Guide*\. 
+ If you did not validate an expression in advance, the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command validates it when you create your scaling policy\. However, there is a possibility that this command might fail to identify the exact cause of the detected errors\. To fix the issues, troubleshoot the errors that you receive in a response from a request to the [get\-metric\-data](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-data.html) command\. You can also troubleshoot the expression from the CloudWatch console\.
+ When you view your **Load** and **Capacity** graphs in the console, the **Capacity** graph might not show any data\. To ensure that the graphs have complete data, make sure that you consistently enable group metrics for your Auto Scaling groups\. For more information, see [Enable Auto Scaling group metrics \(console\)](as-instance-monitoring.md#as-enable-group-metrics)\.
+ The capacity metric specification is only useful for blue/green deployments when you have applications that run in different Auto Scaling groups over their lifetime\. This custom metric lets you provide the total capacity of multiple Auto Scaling groups\. Predictive scaling uses this to show historical data in the **Capacity** graphs in the console\.
+ You must specify `false` for `ReturnData` if `MetricDataQueries` specifies the SEARCH\(\) function on its own without a math function like SUM\(\)\. This is because search expressions might return multiple time series, and a metric specification based on an expression can return only one time series\.
+ All metrics involved in a search expression should be of the same resolution\.

## Limitations<a name="custom-metrics-limitations"></a>
+ You can query data points of up to 10 metrics in one metric specification\.
+ For the purposes of this limit, one expression counts as one metric\.