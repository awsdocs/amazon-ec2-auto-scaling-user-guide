# Scaling based on Amazon SQS<a name="as-using-sqs-queue"></a>

This section shows you how to scale your Auto Scaling group in response to changes in system load in an Amazon Simple Queue Service \(Amazon SQS\) queue\. To learn more about how you can use Amazon SQS, see the [Amazon Simple Queue Service Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/)\.

There are some scenarios where you might think about scaling in response to activity in an Amazon SQS queue\. For example, suppose that you have a web app that lets users upload images and use them online\. In this scenario, each image requires resizing and encoding before it can be published\. The app runs on EC2 instances in an Auto Scaling group, and it's configured to handle your typical upload rates\. Unhealthy instances are terminated and replaced to maintain current instance levels at all times\. The app places the raw bitmap data of the images in an SQS queue for processing\. It processes the images and then publishes the processed images where they can be viewed by users\. The architecture for this scenario works well if the number of image uploads doesn't vary over time\. But if the number of uploads changes over time, you might consider using dynamic scaling to scale the capacity of your Auto Scaling group\.

## Using target tracking with the right metric<a name="scale-sqs-queue-custom-metric"></a>

If you use a target tracking scaling policy based on a custom Amazon SQS queue metric, dynamic scaling can adjust to the demand curve of your application more effectively\. For more information about choosing metrics for target tracking, see [Choosing metrics](as-scaling-target-tracking.md#available-metrics)\.

The issue with using a CloudWatch Amazon SQS metric like `ApproximateNumberOfMessagesVisible` for target tracking is that the number of messages in the queue might not change proportionally to the size of the Auto Scaling group that processes messages from the queue\. That's because the number of messages in your SQS queue does not solely define the number of instances needed\. The number of instances in your Auto Scaling group can be driven by multiple factors, including how long it takes to process a message and the acceptable amount of latency \(queue delay\)\. 

The solution is to use a *backlog per instance* metric with the target value being the *acceptable backlog per instance* to maintain\. You can calculate these numbers as follows:
+ **Backlog per instance**: To calculate your backlog per instance, start with the `ApproximateNumberOfMessages` queue attribute to determine the length of the SQS queue \(number of messages available for retrieval from the queue\)\. Divide that number by the fleet's running capacity, which for an Auto Scaling group is the number of instances in the `InService` state, to get the backlog per instance\.
+ **Acceptable backlog per instance**: To calculate your target value, first determine what your application can accept in terms of latency\. Then, take the acceptable latency value and divide it by the average time that an EC2 instance takes to process a message\. 

To illustrate with an example, let's say that the current `ApproximateNumberOfMessages` is 1500 and the fleet's running capacity is 10\. If the average processing time is 0\.1 seconds for each message and the longest acceptable latency is 10 seconds, then the acceptable backlog per instance is 10 / 0\.1, which equals 100\. This means that 100 is the target value for your target tracking policy\. If the backlog per instance is currently at 150 \(1500 / 10\), your fleet scales out, and it scale out by five instances to maintain proportion to the target value\.

The following procedures demonstrate how to publish the custom metric and create the target tracking scaling policy that configures your Auto Scaling group to scale based on these calculations\.

There are three main parts to this configuration:
+ An Auto Scaling group to manage EC2 instances for the purposes of processing messages from an SQS queue\. 
+ A custom metric to send to Amazon CloudWatch that measures the number of messages in the queue per EC2 instance in the Auto Scaling group\.
+ A target tracking policy that configures your Auto Scaling group to scale based on the custom metric and a set target value\. CloudWatch alarms invoke the scaling policy\. 

The following diagram illustrates the architecture of this configuration\. 

![\[Amazon EC2 Auto Scaling using queues architectural diagram\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/sqs-as-custom-metric-diagram.png)

## Limitations and prerequisites<a name="scale-sqs-queue-limitations"></a>

To use this configuration, you need to be aware of the following limitations:
+ You must use the AWS CLI or AWS SDKs to publish your custom metric to CloudWatch\. You can then monitor your metric with the AWS Management Console\. 
+ After publishing your custom metric, you must use the AWS CLI or AWS SDKs to create a target tracking scaling policy with a customized metric specification\. 

The following sections direct you to use the AWS CLI for the tasks you need to perform\. For example, to get metric data that reflects the present use of the queue, you use the SQS [https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html](https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html) command\. Make sure that you have the CLI [installed](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\. 

Before you begin, you must have an Amazon SQS queue to use\. The following sections assume that you already have a queue \(standard or FIFO\), an Auto Scaling group, and EC2 instances running the application that uses the queue\. For more information about Amazon SQS, see the [Amazon Simple Queue Service Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/)\.

## Configure scaling based on Amazon SQS<a name="scale-sqs-queue-cli"></a>

**Topics**
+ [Step 1: Create a CloudWatch custom metric](#create-sqs-cw-alarms-cli)
+ [Step 2: Create a target tracking scaling policy](#create-sqs-policies-cli)
+ [Step 3: Test your scaling policy](#validate-sqs-scaling-cli)

### Step 1: Create a CloudWatch custom metric<a name="create-sqs-cw-alarms-cli"></a>

A custom metric is defined using a metric name and namespace of your choosing\. Namespaces for custom metrics cannot start with "AWS/"\. For more information about publishing custom metrics, see the [Publish custom metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/publishingMetrics.html) topic in the *Amazon CloudWatch User Guide*\.

Follow this procedure to create the custom metric by first reading information from your AWS account\. Then, calculate the backlog per instance metric, as recommended in an earlier section\. Lastly, publish this number to CloudWatch at a 1\-minute granularity\. Whenever possible, we strongly recommend that you scale on metrics with a 1\-minute granularity to ensure a faster response to changes in system load\. 

**To create a CloudWatch custom metric**

1. Use the SQS [https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html](https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html) command to get the number of messages waiting in the queue \(`ApproximateNumberOfMessages`\)\. 

   ```
   aws sqs get-queue-attributes --queue-url https://sqs.region.amazonaws.com/123456789/MyQueue \
     --attribute-names ApproximateNumberOfMessages
   ```

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to get the running capacity of the group, which is the number of instances in the `InService` lifecycle state\. This command returns the instances of an Auto Scaling group along with their lifecycle state\. 

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names my-asg
   ```

1. Calculate the backlog per instance by dividing the approximate number of messages available for retrieval from the queue by the fleet's running capacity\. 

1. Publish the results at a 1\-minute granularity as a CloudWatch custom metric\. 

   Here is an example CLI [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-data.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-data.html) command\.

   ```
   aws cloudwatch put-metric-data --metric-name MyBacklogPerInstance --namespace MyNamespace \
     --unit None --value 20 --dimensions MyOptionalMetricDimensionName=MyOptionalMetricDimensionValue
   ```

After your application is emitting the desired metric, the data is sent to CloudWatch\. The metric is visible in the CloudWatch console\. You can access it by logging into the AWS Management Console and navigating to the CloudWatch page\. Then, view the metric by navigating to the metrics page or by searching for it using the search box\. For information about viewing metrics, see [View available metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/viewing_metrics_with_cloudwatch.html) in the *Amazon CloudWatch User Guide*\.

### Step 2: Create a target tracking scaling policy<a name="create-sqs-policies-cli"></a>

After publishing your custom metric, create a target tracking scaling policy with a customized metric specification\. 

**To create a target tracking scaling policy**

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

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command, along with the `config.json` file that you created in the previous step, to create your scaling policy\.

   ```
   aws autoscaling put-scaling-policy --policy-name sqs100-target-tracking-scaling-policy \
     --auto-scaling-group-name my-asg --policy-type TargetTrackingScaling \
     --target-tracking-configuration file://~/config.json
   ```

   This creates two alarms: one for scaling out and one for scaling in\. It also returns the Amazon Resource Name \(ARN\) of the policy that is registered with CloudWatch, which CloudWatch uses to invoke scaling whenever the metric is in breach\. 

### Step 3: Test your scaling policy<a name="validate-sqs-scaling-cli"></a>

After your setup is complete, verify that your scaling policy is working\. You can test it by increasing the number of messages in your SQS queue and then verifying that your Auto Scaling group has launched an additional EC2 instance\. You can also test it by decreasing the number of messages in your SQS queue and then verifying that the Auto Scaling group has terminated an EC2 instance\.

**To test the scale\-out function**

1. Follow the steps in [Tutorial: Sending a message to an Amazon SQS queue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-send-message.html) to add messages to your queue\. Make sure that you have increased the number of messages in the queue so that the backlog per instance metric exceeds the target value\.

   It can take a few minutes for your changes to trigger the CloudWatch alarm\. 

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the group has launched an instance\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

**To test the scale\-in function**

1. Follow the steps in [Tutorial: Sending a message to an Amazon SQS queue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-send-message.html) to remove messages from the queue\. Make sure that you have decreased the number of messages in the queue so that the backlog per instance metric is below the target value\.

   It can take a few minutes for your changes to trigger the CloudWatch alarm\. 

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the group has terminated an instance\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```