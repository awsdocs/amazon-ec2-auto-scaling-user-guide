# Monitoring Your Auto Scaling Instances and Groups<a name="as-monitoring-features"></a>

You can use the following features to monitor your Auto Scaling instances and groups\.

**Health checks**  
Auto Scaling periodically performs health checks on the instances in your Auto Scaling group and identifies any instances that are unhealthy\. You can configure Auto Scaling to determine the health status of an instance using Amazon EC2 status checks, Elastic Load Balancing health checks, or custom health checks\. For more information, see [Health Checks for Auto Scaling Instances](healthcheck.md)\.

**CloudWatch metrics**  
Auto Scaling publishes data points to Amazon CloudWatch about your Auto Scaling groups\. CloudWatch enables you to retrieve statistics about those data points as an ordered set of time\-series data, known as *metrics*\. You can use these metrics to verify that your system is performing as expected\. For more information, see [Monitoring Your Auto Scaling Groups and Instances Using Amazon CloudWatch](as-instance-monitoring.md)\.

**CloudWatch Events**  
Auto Scaling can submit events to Amazon CloudWatch Events when your Auto Scaling groups launch or terminate instances, or when a lifecycle action occurs\. This enables you to invoke a Lambda function when the event occurs\. For more information, see [Getting CloudWatch Events When Your Auto Scaling Group Scales](cloud-watch-events.md)\.

**SNS notifications**  
Auto Scaling can send Amazon SNS notifications when your Auto Scaling groups launch or terminate instances\. For more information, see [Getting SNS Notifications When Your Auto Scaling Group Scales](ASGettingNotifications.md)\.

**CloudTrail logs**  
AWS CloudTrail enables you to keep track of the calls made to the Auto Scaling API by or on behalf of your AWS account\. CloudTrail stores the information in log files in the Amazon S3 bucket that you specify\. You can use these log files to monitor activity of your Auto Scaling groups by determining which requests were made, the source IP addresses where the requests came from, who made the request, when the request was made, and so on\. For more information, see [Logging Amazon EC2 Auto Scaling API Calls By Using AWS CloudTrail](logging-using-cloudtrail.md)\.