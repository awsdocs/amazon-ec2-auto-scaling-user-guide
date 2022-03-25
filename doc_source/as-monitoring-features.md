# Monitoring your Auto Scaling instances and groups<a name="as-monitoring-features"></a>

Monitoring is an important part of maintaining the reliability, availability, and performance of Amazon EC2 Auto Scaling and your Amazon Web Services Cloud solutions\. AWS provides the following monitoring tools to watch Amazon EC2 Auto Scaling, report when something is wrong, and take automatic actions when appropriate:

**Health Checks**  
Amazon EC2 Auto Scaling periodically performs health checks on the instances in your Auto Scaling group\. If an instance does not pass its health check, it is marked unhealthy and will be terminated while Amazon EC2 Auto Scaling launches a new instance in replacement\. For more information, see [Health checks for Auto Scaling instances](healthcheck.md)\.

**Capacity Rebalancing**  
You can enable Capacity Rebalancing on new and existing Auto Scaling groups when using Spot Instances\. When you turn on Capacity Rebalancing, Amazon EC2 Auto Scaling attempts to launch a Spot Instance whenever Amazon EC2 notifies that a Spot Instance is at an elevated risk of interruption\. After launching a new instance, it then terminates an old instance\. For more information, see [Amazon EC2 Auto Scaling Capacity Rebalancing](ec2-auto-scaling-capacity-rebalancing.md)\. 

**AWS Health Dashboard**  
The AWS Health Dashboard \(PHD\) displays information, and also provides notifications that are triggered by changes in the health of Amazon Web Services resources\. The information is presented in two ways: on a dashboard that shows recent and upcoming events organized by category, and in a full event log that shows all events from the past 90 days\. For more information, see [AWS Health Dashboard notifications for Amazon EC2 Auto Scaling](monitoring-personal-health-dashboard.md)\.

**CloudWatch Alarms**  
To detect unhealthy application behavior, CloudWatch helps you by automatically monitoring certain metrics for your Amazon Web Services resources\. You can configure a CloudWatch alarm and set up an Amazon SNS notification that sends an email when a metric's value is not what you expect or when certain anomalies are detected\. For example, you can be notified when network activity is suddenly higher or lower than a metric's expected value\. For more information, see [Monitoring CloudWatch metrics for your Auto Scaling groups and instances](as-instance-monitoring.md)\.

**CloudWatch Dashboards**  
CloudWatch dashboards are customizable home pages in the CloudWatch console\. You can use these pages to monitor your resources in a single view, even including resources that are spread across different Regions\. You can use CloudWatch dashboards to create customized views of the metrics and alarms for your Amazon Web Services resources\. For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/)\.

**CloudTrail Logs**  
AWS CloudTrail enables you to track the calls made to the Amazon EC2 Auto Scaling API by or on behalf of your Amazon Web Services account\. CloudTrail stores the information in log files in the Amazon S3 bucket that you specify\. You can use these log files to monitor activity of your Auto Scaling groups\. Logs include which requests were made, the source IP addresses where the requests came from, who made the request, when the request was made, and so on\. For more information, see [Logging Amazon EC2 Auto Scaling API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

**CloudWatch Logs**  
CloudWatch Logs enable you to monitor, store, and access your log files from Amazon EC2 instances, CloudTrail, and other sources\. CloudWatch Logs can monitor information in the log files and notify you when certain thresholds are met\. You can also archive your log data in highly durable storage\. For more information, see the [Amazon CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)\.

**Amazon Simple Notification Service Notifications**  
You can configure Auto Scaling groups to send Amazon SNS notifications when Amazon EC2 Auto Scaling launches or terminates instances\. For more information, see [Getting Amazon SNS notifications when your Auto Scaling group scales](ASGettingNotifications.md)\. 

**EventBridge**  
Amazon EventBridge, formerly called CloudWatch Events, delivers a near real\-time stream of system events that describe changes in Amazon Web Services resources\. EventBridge enables automated event\-driven computing, as you can write rules that watch for certain events and trigger automated actions in other Amazon Web Services when these events happen\. For example, you can use EventBridge to set up a target to invoke a Lambda function when your Auto Scaling group scales or when a lifecycle action occurs\. You can also receive a two\-minute warning when Spot Instances are about to be reclaimed by Amazon EC2\. For information about capturing Amazon EC2 Auto Scaling emitted events in EventBridge, see [Using Amazon EC2 Auto Scaling with EventBridge](cloud-watch-events.md)\. 