# Monitoring Your Auto Scaling Instances and Groups<a name="as-monitoring-features"></a>

Monitoring is an important part of maintaining the reliability, availability, and performance of Amazon EC2 Auto Scaling and your AWS solutions\. You should collect monitoring data from all parts of your AWS solution so that you can more easily debug a multi\-point failure if one occurs\. AWS provides monitoring tools to watch Amazon EC2 Auto Scaling, report when something is wrong, and take automatic actions when appropriate\.

**Health Checks**  
Amazon EC2 Auto Scaling periodically performs health checks on the instances in your Auto Scaling group and identifies any instances that are unhealthy\. You can configure Auto Scaling groups to determine the health status of an instance using Amazon EC2 status checks, Elastic Load Balancing health checks, or custom health checks\. For more information, see [Health Checks for Auto Scaling Instances](healthcheck.md)\.

**Amazon Simple Notification Service Notifications**  
You can configure Auto Scaling groups to send Amazon SNS notifications when Amazon EC2 Auto Scaling launches or terminates instances\. For more information, see [Getting Amazon SNS Notifications When Your Auto Scaling Group Scales](ASGettingNotifications.md)\.

**CloudWatch Events**  
Amazon EC2 Auto Scaling can submit events to Amazon CloudWatch Events when your Auto Scaling groups launch or terminate instances, or when a lifecycle action occurs\. This enables you to trigger automated actions in other AWS services when these events happen\. For more information, see [Getting CloudWatch Events When Your Auto Scaling Group Scales](cloud-watch-events.md)\.  
You can also write rules that trigger on API calls made by Amazon EC2 Auto Scaling\. For more information, see [Creating a CloudWatch Events Rule That Triggers on an AWS API Call Using AWS CloudTrail](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/Create-CloudWatch-Events-CloudTrail-Rule.html) in the *Amazon CloudWatch Events User Guide*\. 

**CloudWatch Alarms**  
To detect unhealthy application behavior, CloudWatch helps you by automatically monitoring certain metrics for your AWS resources\. You can configure a CloudWatch alarm and set up an Amazon SNS notification that sends an email when a metric’s value is not what you expect or when certain anomalies are detected\. For example, you can be notified when network activity is suddenly higher or lower than a metric's expected value\. For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/)\.

**CloudWatch Dashboards**  
CloudWatch dashboards are customizable home pages in the CloudWatch console\. You can use these pages to monitor your resources in a single view, even including resources that are spread across different Regions\. You can use CloudWatch dashboards to create customized views of the metrics and alarms for your AWS resources\. For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/)\.

**CloudTrail Logs**  
AWS CloudTrail enables you to track the calls made to the Amazon EC2 Auto Scaling API by or on behalf of your AWS account\. CloudTrail stores the information in log files in the Amazon S3 bucket that you specify\. You can use these log files to monitor activity of your Auto Scaling groups\. Logs include which requests were made, the source IP addresses where the requests came from, who made the request, when the request was made, and so on\. For more information, see [Logging Amazon EC2 Auto Scaling API Calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

**CloudWatch Logs**  
CloudWatch Logs enable you to monitor, store, and access your log files from Amazon EC2 instances, CloudTrail, and other sources\. CloudWatch Logs can monitor information in the log files and notify you when certain thresholds are met\. You can also archive your log data in highly durable storage\. For more information, see the [Amazon CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)\.

**AWS Personal Health Dashboard**  
The Personal Health Dashboard \(PHD\) displays information, and also provides notifications that are triggered by changes in the health of AWS resources\. The information is presented in two ways: on a dashboard that shows recent and upcoming events organized by category, and in a full event log that shows all events from the past 90 days\. For more information, see [Personal Health Dashboard Notifications for Amazon EC2 Auto Scaling](monitoring-personal-health-dashboard.md)\.