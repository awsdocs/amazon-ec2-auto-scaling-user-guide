# Monitoring Your Auto Scaling Instances and Groups<a name="as-monitoring-features"></a>

Monitoring is an important part of maintaining the reliability, availability, and performance of Amazon EC2 Auto Scaling and your AWS solutions\. AWS provides monitoring tools to watch Amazon EC2 Auto Scaling, report when something is wrong, and take automatic actions when appropriate\.

You can use the following features to monitor your Auto Scaling instances and groups\.

**Health Checks**  
Amazon EC2 Auto Scaling periodically performs health checks on the instances in your Auto Scaling group and identifies any instances that are unhealthy\. You can configure Auto Scaling groups to determine the health status of an instance using Amazon EC2 status checks, Elastic Load Balancing health checks, or custom health checks\. For more information, see [Health Checks for Auto Scaling Instances](healthcheck.md)\.

**Amazon Simple Notification Service Notifications**  
You can configure Auto Scaling groups to send Amazon SNS notifications when Amazon EC2 Auto Scaling launches or terminates instances\. For more information, see [Getting Amazon SNS Notifications When Your Auto Scaling Group Scales](ASGettingNotifications.md)\.

**CloudWatch Events**  
Amazon EC2 Auto Scaling can submit events to Amazon CloudWatch Events when your Auto Scaling groups launch or terminate instances, or when a lifecycle action occurs\. This enables you to invoke a Lambda function when the event occurs\. For more information, see [Getting CloudWatch Events When Your Auto Scaling Group Scales](cloud-watch-events.md)\.

**Amazon CloudWatch Alarms**  
Amazon EC2 Auto Scaling publishes data points to Amazon CloudWatch about your Auto Scaling groups\. CloudWatch enables you to retrieve statistics about those data points as an ordered set of time series data, known as *metrics*\. Using CloudWatch alarms, you watch a single metric\. If the metric exceeds a given threshold for a specified number of evaluation periods, a notification is sent to an Amazon SNS topic\. For more information, see [Monitoring Your Auto Scaling Groups and Instances Using Amazon CloudWatch](as-instance-monitoring.md)\.

**Amazon CloudWatch Dashboards**  
CloudWatch dashboards are customizable home pages in the CloudWatch console\. You can use these pages to monitor your resources in a single view, even including resources that are spread across different Regions\. You can use CloudWatch dashboards to create customized views of the metrics and alarms for your AWS resources\. For more information, see [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/)\.

**CloudTrail logs**  
AWS CloudTrail enables you to track the calls made to the Amazon EC2 Auto Scaling API by or on behalf of your AWS account\. CloudTrail stores the information in log files in the Amazon S3 bucket that you specify\. You can use these log files to monitor activity of your Auto Scaling groups\. Logs include which requests were made, the source IP addresses where the requests came from, who made the request, when the request was made, and so on\. For more information, see [Logging Amazon EC2 Auto Scaling API Calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

**Amazon CloudWatch Logs**  
Amazon CloudWatch Logs enable you to monitor, store, and access your log files from Amazon EC2 instances, CloudTrail, and other sources\. CloudWatch Logs can monitor information in the log files and notify you when certain thresholds are met\. You can also archive your log data in highly durable storage\. For more information, see the [Amazon CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)\.