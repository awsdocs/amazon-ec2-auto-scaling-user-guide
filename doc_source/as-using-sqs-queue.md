# Scaling based on Amazon SQS<a name="as-using-sqs-queue"></a>

**Important**  
The following information and steps shows you how to calculate the Amazon SQS queue backlog per instance using the `ApproximateNumberOfMessages` queue attribute before publishing it as a custom metric to CloudWatch\. However, you can now save the cost and effort put into publishing your own metric by using metric math\. For more information, see [Create a target tracking scaling policy for Amazon EC2 Auto Scaling using metric math](ec2-auto-scaling-target-tracking-metric-math.md)\.

This section shows you how to scale your Auto Scaling group in response to changes in system load in an Amazon Simple Queue Service \(Amazon SQS\) queue\. To learn more about how you can use Amazon SQS, see the [Amazon Simple Queue Service Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/)\.

There are some scenarios where you might think about scaling in response to activity in an Amazon SQS queue\. For example, suppose that you have a web app that lets users upload images and use them online\. In this scenario, each image requires resizing and encoding before it can be published\. The app runs on EC2 instances in an Auto Scaling group, and it's configured to handle your typical upload rates\. Unhealthy instances are terminated and replaced to maintain current instance levels at all times\. The app places the raw bitmap data of the images in an SQS queue for processing\. It processes the images and then publishes the processed images where they can be viewed by users\. The architecture for this scenario works well if the number of image uploads doesn't vary over time\. But if the number of uploads changes over time, you might consider using dynamic scaling to scale the capacity of your Auto Scaling group\.

**Topics**
+ [Use target tracking with the right metric](#scale-sqs-queue-custom-metric)
+ [Limitations and prerequisites](#scale-sqs-queue-limitations)
+ [Configure scaling based on Amazon SQS](#scale-sqs-queue-cli)
+ [Amazon SQS and instance scale\-in protection](#scale-sqs-queue-scale-in-protection)

## Use target tracking with the right metric<a name="scale-sqs-queue-custom-metric"></a>

If you use a target tracking scaling policy based on a custom Amazon SQS queue metric, dynamic scaling can adjust to the demand curve of your application more effectively\. For more information about choosing metrics for target tracking, see [Choose metrics](as-scaling-target-tracking.md#target-tracking-choose-metrics)\.

The issue with using a CloudWatch Amazon SQS metric like `ApproximateNumberOfMessagesVisible` for target tracking is that the number of messages in the queue might not change proportionally to the size of the Auto Scaling group that processes messages from the queue\. That's because the number of messages in your SQS queue does not solely define the number of instances needed\. The number of instances in your Auto Scaling group can be driven by multiple factors, including how long it takes to process a message and the acceptable amount of latency \(queue delay\)\. 

The solution is to use a *backlog per instance* metric with the target value being the *acceptable backlog per instance* to maintain\. You can calculate these numbers as follows:
+ **Backlog per instance**: To calculate your backlog per instance, start with the `ApproximateNumberOfMessages` queue attribute to determine the length of the SQS queue \(number of messages available for retrieval from the queue\)\. Divide that number by the fleet's running capacity, which for an Auto Scaling group is the number of instances in the `InService` state, to get the backlog per instance\.
+ **Acceptable backlog per instance**: To calculate your target value, first determine what your application can accept in terms of latency\. Then, take the acceptable latency value and divide it by the average time that an EC2 instance takes to process a message\. 

As an example, let's say that you currently have an Auto Scaling group with 10 instances and the number of visible messages in the queue \(`ApproximateNumberOfMessages`\) is 1500\. If the average processing time is 0\.1 seconds for each message and the longest acceptable latency is 10 seconds, then the acceptable backlog per instance is 10 / 0\.1, which equals 100 messages\. This means that 100 is the target value for your target tracking policy\. When the backlog per instance reaches the target value, a scale\-out event will happen\. Because the backlog per instance is already 150 messages \(1500 messages / 10 instances\), your group scales out, and it scales out by five instances to maintain proportion to the target value\.

The following procedures demonstrate how to publish the custom metric and create the target tracking scaling policy that configures your Auto Scaling group to scale based on these calculations\.

**Important**  
Remember, to reduce costs, use metric math instead\. For more information, see [Create a target tracking scaling policy for Amazon EC2 Auto Scaling using metric math](ec2-auto-scaling-target-tracking-metric-math.md)\.

There are three main parts to this configuration:
+ An Auto Scaling group to manage EC2 instances for the purposes of processing messages from an SQS queue\. 
+ A custom metric to send to Amazon CloudWatch that measures the number of messages in the queue per EC2 instance in the Auto Scaling group\.
+ A target tracking policy that configures your Auto Scaling group to scale based on the custom metric and a set target value\. CloudWatch alarms invoke the scaling policy\. 

The following diagram illustrates the architecture of this configuration\. 

![\[Amazon EC2 Auto Scaling using queues architectural diagram\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/sqs-as-custom-metric-diagram.png)

## Limitations and prerequisites<a name="scale-sqs-queue-limitations"></a>

To use this configuration, you need to be aware of the following limitations:
+ You must use the AWS CLI or an SDK to publish your custom metric to CloudWatch\. You can then monitor your metric with the AWS Management Console\. 
+ The Amazon EC2 Auto Scaling console does not support target tracking scaling policies that use custom metrics\. You must use the AWS CLI or an SDK to specify a custom metric for your scaling policy\.

The following sections direct you to use the AWS CLI for the tasks you need to perform\. For example, to get metric data that reflects the present use of the queue, you use the SQS [get\-queue\-attributes](https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html) command\. Make sure that you have the CLI [installed](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\. 

Before you begin, you must have an Amazon SQS queue to use\. The following sections assume that you already have a queue \(standard or FIFO\), an Auto Scaling group, and EC2 instances running the application that uses the queue\. For more information about Amazon SQS, see the [Amazon Simple Queue Service Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/)\.

## Configure scaling based on Amazon SQS<a name="scale-sqs-queue-cli"></a>

**Topics**
+ [Step 1: Create a CloudWatch custom metric](#create-sqs-cw-alarms-cli)
+ [Step 2: Create a target tracking scaling policy](#create-sqs-policies-cli)
+ [Step 3: Test your scaling policy](#validate-sqs-scaling-cli)

### Step 1: Create a CloudWatch custom metric<a name="create-sqs-cw-alarms-cli"></a>

A custom metric is defined using a metric name and namespace of your choosing\. Namespaces for custom metrics cannot start with `AWS/`\. For more information about publishing custom metrics, see the [Publish custom metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/publishingMetrics.html) topic in the *Amazon CloudWatch User Guide*\.

Follow this procedure to create the custom metric by first reading information from your AWS account\. Then, calculate the backlog per instance metric, as recommended in an earlier section\. Lastly, publish this number to CloudWatch at a 1\-minute granularity\. Whenever possible, we strongly recommend that you scale on metrics with a 1\-minute granularity to ensure a faster response to changes in system load\. 

**To create a CloudWatch custom metric \(AWS CLI\)**

1. Use the SQS [get\-queue\-attributes](https://docs.aws.amazon.com/cli/latest/reference/sqs/get-queue-attributes.html) command to get the number of messages waiting in the queue \(`ApproximateNumberOfMessages`\)\. 

   ```
   aws sqs get-queue-attributes --queue-url https://sqs.region.amazonaws.com/123456789/MyQueue \
     --attribute-names ApproximateNumberOfMessages
   ```

1. Use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to get the running capacity of the group, which is the number of instances in the `InService` lifecycle state\. This command returns the instances of an Auto Scaling group along with their lifecycle state\. 

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names my-asg
   ```

1. Calculate the backlog per instance by dividing the approximate number of messages available for retrieval from the queue by the group's running capacity\. 

1. Create a script that runs every minute to retrieve the backlog per instance value and publish it to a CloudWatch custom metric\. When you publish a custom metric, you specify the metric's name, namespace, unit, value, and zero or more dimensions\. A dimension consists of a dimension name and a dimension value\.

   To publish your custom metric, replace placeholder values in *italics* with your preferred metric name, the metric's value, a namespace \(as long as it doesn’t begin with "`AWS`"\), and dimensions \(optional\), and then run the following [put\-metric\-data](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-data.html) command\. 

   ```
   aws cloudwatch put-metric-data --metric-name MyBacklogPerInstance --namespace MyNamespace \
     --unit None --value 20 --dimensions MyOptionalMetricDimensionName=MyOptionalMetricDimensionValue
   ```

After your application is emitting the desired metric, the data is sent to CloudWatch\. The metric is visible in the CloudWatch console\. You can access it by logging into the AWS Management Console and navigating to the CloudWatch page\. Then, view the metric by navigating to the metrics page or by searching for it using the search box\. For information about viewing metrics, see [View available metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/viewing_metrics_with_cloudwatch.html) in the *Amazon CloudWatch User Guide*\.

### Step 2: Create a target tracking scaling policy<a name="create-sqs-policies-cli"></a>

The metric you created can now be added to a target tracking scaling policy\.

**To create a target tracking scaling policy \(AWS CLI\)**

1. Use the following `cat` command to store a target value for your scaling policy and a customized metric specification in a JSON file named `config.json` in your home directory\. Replace placeholder values in *italics* with your own values\. For the `TargetValue`, calculate the acceptable backlog per instance metric and enter it here\. To calculate this number, decide on a normal latency value and divide it by the average time that it takes to process a message, as described in an earlier section\. 

   If you didn't specify any dimensions for the metric you created in step 1, don't include any dimensions in the customized metric specification\.

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

1. Use the [put\-scaling\-policy](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command, along with the `config.json` file that you created in the previous step, to create your scaling policy\.

   ```
   aws autoscaling put-scaling-policy --policy-name sqs100-target-tracking-scaling-policy \
     --auto-scaling-group-name my-asg --policy-type TargetTrackingScaling \
     --target-tracking-configuration file://~/config.json
   ```

   This creates two alarms: one for scaling out and one for scaling in\. It also returns the Amazon Resource Name \(ARN\) of the policy that is registered with CloudWatch, which CloudWatch uses to invoke scaling whenever the metric threshold is in breach\. 

### Step 3: Test your scaling policy<a name="validate-sqs-scaling-cli"></a>

After your setup is complete, verify that your scaling policy is working\. You can test it by increasing the number of messages in your SQS queue and then verifying that your Auto Scaling group has launched an additional EC2 instance\. You can also test it by decreasing the number of messages in your SQS queue and then verifying that the Auto Scaling group has terminated an EC2 instance\.

**To test the scale\-out function**

1. Follow the steps in [Sending messages to a queue \(console\)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-using-send-messages.html) to add messages to your queue\. Make sure that you have increased the number of messages in the queue so that the backlog per instance metric exceeds the target value\.

   It can take a few minutes for your changes to invoke the alarm\.

1. Use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the group has launched an instance\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

**To test the scale\-in function**

1. Follow the steps in [Receiving and deleting messages \(console\)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-using-receive-delete-message.html) to delete messages from the queue\. Make sure that you have decreased the number of messages in the queue so that the backlog per instance metric is below the target value\.

   It can take a few minutes for your changes to invoke the alarm\.

1. Use the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the group has terminated an instance\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

## Amazon SQS and instance scale\-in protection<a name="scale-sqs-queue-scale-in-protection"></a>

Messages that have not been processed at the time an instance is terminated are returned to the SQS queue where they can be processed by another instance that is still running\. For applications where long running tasks are performed, you can optionally use instance scale\-in protection to have control over which queue workers are terminated when your Auto Scaling group scales in\.

The following pseudocode shows one way to protect long\-running, queue\-driven worker processes from scale\-in termination\.

```
while (true)
{
  SetInstanceProtection(False);
  Work = GetNextWorkUnit();
  SetInstanceProtection(True);
  ProcessWorkUnit(Work);
  SetInstanceProtection(False);
}
```

For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.