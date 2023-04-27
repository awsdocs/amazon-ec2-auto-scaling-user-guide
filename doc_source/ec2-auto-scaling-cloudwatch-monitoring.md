# Monitor CloudWatch metrics for your Auto Scaling groups and instances<a name="ec2-auto-scaling-cloudwatch-monitoring"></a>

*Metrics* are the fundamental concept in Amazon CloudWatch\. A metric represents a time\-ordered set of data points that are published to CloudWatch\. Think of a metric as a variable to monitor, and the data points as representing the values of that variable over time\. You can use these metrics to verify that your system is performing as expected\. 

Amazon EC2 Auto Scaling metrics that collect information about Auto Scaling groups are in the `AWS/AutoScaling` namespace\. Amazon EC2 instance metrics that collect CPU and other usage data from Auto Scaling instances are in the `AWS/EC2` namespace\. 

The Amazon EC2 Auto Scaling console displays a series of graphs for the group metrics and the aggregated instance metrics for the group\. Depending on your needs, you might prefer to access data for your Auto Scaling groups and instances from Amazon CloudWatch instead of the Amazon EC2 Auto Scaling console\.

For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.

**Topics**
+ [View monitoring graphs in the Amazon EC2 Auto Scaling console](viewing-monitoring-graphs.md)
+ [Amazon CloudWatch metrics for Amazon EC2 Auto Scaling](ec2-auto-scaling-metrics.md)
+ [Configure monitoring for Auto Scaling instances](enable-as-instance-metrics.md)