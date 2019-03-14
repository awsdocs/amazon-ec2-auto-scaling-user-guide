# Scaling Based on Amazon SQS<a name="as-using-sqs-queue"></a>

This section shows you how to scale your Auto Scaling group in response to changing demand from an Amazon Simple Queue Service \(Amazon SQS\) queue\. Amazon SQS offers a secure, durable, and available hosted queue that lets you integrate and decouple distributed software systems and components\. If you're not familiar with Amazon SQS, see the [Amazon Simple Queue Service Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/) for more information\.

There are several scenarios where you might think about scaling in response to activity in an Amazon SQS queue\. Suppose that you have a web app that lets users upload images and use them online\. Each image requires resizing and encoding before it can be published\. The app runs on EC2 instances in an Auto Scaling group that is configured to handle your typical upload rates\. Unhealthy instances are terminated and replaced to maintain current instance levels at all times\. The app places the raw bitmap data of the images in an Amazon SQS queue for processing\. It processes the images and then publishes the processed images where they can be viewed by users\. 

This architecture works well if the number of image uploads doesn't vary over time\. What happens if your upload levels change? If your uploads increase and decrease on a predictable schedule, you can specify the time and date to perform scaling activities\. For more information, see [Scheduled Scaling for Amazon EC2 Auto Scaling](schedule_time.md)\. A more dynamic way to scale your Auto Scaling group, scaling by policy, lets you define parameters that control the scaling process\. For example, you can create a policy that calls for enlarging your fleet of EC2 instances whenever the average number of uploads reaches a certain level\. This is useful for scaling in response to changing conditions, when you don't know when those conditions will change\.

There are three main parts to this configuration:
+ An Auto Scaling group to manage EC2 instances for the purposes of processing messages from an SQS queue\. 
+ A custom metric to send to Amazon CloudWatch that measures the number of messages in the queue per EC2 instance in the Auto Scaling group\.
+ A target tracking policy that configures your Auto Scaling group to scale based on the custom metric and a set target value\. CloudWatch alarms invoke the scaling policy\. 

The following diagram illustrates the architecture of this configuration\. 

![\[Amazon EC2 Auto Scaling using queues architectural diagram\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/sqs-as-custom-metric-diagram.png)

## Choosing an Effective Metric and Target Value<a name="scale-sqs-queue-custom-metric"></a>

The number of messages in your Amazon SQS queue does not solely define the number of instances needed\. In fact, the number of instances in the fleet can be driven by multiple factors, including how long it takes to process a message and the acceptable amount of latency \(queue delay\)\.

The solution is to use a *backlog per instance* metric with the target value being the *acceptable backlog per instance* to maintain\. You can calculate these numbers as follows:
+ **Backlog per instance**: To determine your backlog per instance, start with the Amazon SQS metric `ApproximateNumberOfMessages` to determine the length of the SQS queue \(number of messages available for retrieval from the queue\)\. Divide that number by the fleet's running capacity, which for an Auto Scaling group is the number of instances in the `InService` state, to get the backlog per instance\.
+ **Acceptable backlog per instance**: To determine your target value, first calculate what your application can accept in terms of latency\. Then, take the acceptable latency value and divide it by the average time that an EC2 instance takes to process a message\. 

To illustrate with an example, the current `ApproximateNumberOfMessages` is 1500 and the fleet's running capacity is 10\. If the average processing time is 0\.1 seconds for each message and the longest acceptable latency is 10 seconds, then the acceptable backlog per instance is 10 / 0\.1, which equals 100\. This means that 100 is the target value for your target tracking policy\. Because the backlog per instance is currently at 150 \(1500 / 10\), your fleet scales out by five instances to maintain proportion to the target value\. 

The following examples create the custom metric and target tracking scaling policy that configures your Auto Scaling group to scale based on these calculations\.

## Scaling with Amazon SQS Using the AWS CLI<a name="scale-sqs-queue-cli"></a>

The following section shows you how to set up automatic scaling for an SQS queue using the AWS CLI\. The procedures assume that you already have a queue \(standard or FIFO\), an Auto Scaling group, and EC2 instances running the application that uses the queue\. 

### Step 1: Create the CloudWatch Custom Metric<a name="create-sqs-cw-alarms-cli"></a>

Create the custom calculated metric by first reading metrics from your AWS account\. Next, calculate the backlog per instance metric recommended in an earlier section\. Lastly, publish this number to CloudWatch at a 1\-minute granularity\. 

Wherever possible, you should scale on EC2 instance metrics with a 1\-minute frequency \(also known as detailed monitoring\) because that ensures a faster response to utilization changes\. Scaling on metrics with a 5\-minute frequency can result in slower response times and scaling on stale metric data\. By default, EC2 instances are enabled for basic monitoring, which means metric data for instances is available at 5\-minute intervals\. You can enable detailed monitoring to get metric data for instances at a 1\-minute frequency\. For more information, see [Monitoring Your Auto Scaling Groups and Instances Using Amazon CloudWatch](as-instance-monitoring.md)\.

**To create the CloudWatch custom metric**

1. Use the SQS [https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html](https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html) command to get the number of messages waiting in the queue \(`ApproximateNumberOfMessages`\): 

   ```
   aws sqs get-queue-attributes --queue-url https://sqs.us-west-2.amazonaws.com/123456789/MyQueue --attribute-names ApproximateNumberOfMessages
   ```

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to get the running capacity of the group, which is the number of instances in the `InService` lifecycle state\. This command returns the instances of an Auto Scaling group along with their lifecycle state\. 

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names MyAutoScalingGroup
   ```

1. Calculate the backlog per instance by dividing the `ApproximateNumberOfMessages` metric by the fleet's running capacity metric\. 

1. Publish the results at a 1\-minute granularity as a CloudWatch custom metric using the AWS CLI or an API\. A custom metric is defined using a metric name and namespace of your choosing\. Namespaces for custom metrics cannot start with "AWS/"\. For more information about publishing custom metrics, see the [Publish Custom Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/publishingMetrics.html) topic in the *Amazon CloudWatch User Guide*\.

   Here is an example CLI [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-data.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-data.html) command:

   ```
   aws cloudwatch put-metric-data --metric-name MyBacklogPerInstance --namespace MyNamespace --unit None --value 20 --dimensions MyOptionalMetricDimensionName=MyOptionalMetricDimensionValue
   ```

After your application is emitting the desired metric, the data is sent to CloudWatch\. The metrics are visible in the CloudWatch console\. You can access them by logging into the AWS Management Console and navigating to the CloudWatch page\. You can view the metric by navigating to the metrics page or searching for your metric using the search box\. For help viewing metrics, see [View Available Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/viewing_metrics_with_cloudwatch.html) in the *Amazon CloudWatch User Guide*\.

### Step 2: Create the Scaling Policies<a name="create-sqs-policies-cli"></a>

Next, create a target tracking scaling policy that tells the Auto Scaling group what to do when conditions change\.

**To create scaling policies**

1. Use the following command to specify a target value for your scaling policy in a `config.json` file in your home directory\. 

   For the `TargetValue`, calculate the acceptable backlog per instance metric and enter it here\. To calculate this number, decide on a normal latency value and divide it by the average time that it takes to process a message\. 

   ```
   $ cat ~/config.json
   {
      "TargetValue":100,
      "CustomizedMetricSpecification":{
         "MetricName":"MyBacklogPerInstance",
         "Namespace":"MyNamespace",
         "Dimensions":[
            {
               "Name":"MyOptionalMetricDimensionName",
               "Value":"MyOptionalMetricDimensionValue"
            }
         ],
         "Statistic":"Average",
         "Unit":"None"
      }
   }
   ```

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command, along with the `config.json` file that you created in the previous step, to create your scaling policy:

   ```
   aws autoscaling put-scaling-policy --policy-name MyScalingPolicy --auto-scaling-group-name MyAutoScalingGroup --policy-type TargetTrackingScaling --target-tracking-configuration file://~/config.json
   ```

   This creates two alarms: one for scaling out and one for scaling in\. It also returns the Amazon Resource Name \(ARN\) of the policy that is registered with CloudWatch, which CloudWatch uses to invoke scaling whenever the metric is in breach\. 

### Step 3: Test Your Scaling Policies<a name="validate-sqs-scaling-cli"></a>

You can test your scale\-out policy by increasing the number of messages in your SQS queue and then verifying that your Auto Scaling group has launched an additional EC2 instance\. Similarly, you can test your scale\-in policy by decreasing the number of messages in your SQS queue and then verifying that the Auto Scaling group has terminated an EC2 instance\.

**To test the scale\-out policy**

1. Follow the steps in [Tutorial: Sending a Message to an Amazon SQS Queue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-send-message.html) to add messages to your queue\. Make sure that you have increased the number of messages in the queue so that the backlog per instance metric exceeds the target value\.

   It can take a few minutes for your changes to trigger the CloudWatch alarm\. 

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the group has launched an instance:

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name MyAutoScalingGroup
   ```

**To test the scale\-in policy**

1. Follow the steps in [Tutorial: Sending a Message to an Amazon SQS Queue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-send-message.html) to remove messages from the queue\. Make sure that you have decreased the number of messages in the queue so that the backlog per instance metric is below the target value\.

   It can take a few minutes for your changes to trigger the CloudWatch alarm\. 

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the group has terminated an instance:

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name MyAutoScalingGroup
   ```

For more information about target tracking, see [Target Tracking Scaling Policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)\.