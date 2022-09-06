# Configure monitoring for Auto Scaling instances<a name="enable-as-instance-metrics"></a>

Amazon EC2 collects and processes raw data from instances into readable, near real\-time metrics that describe the CPU and other usage data for your Auto Scaling group\. You can configure the interval for monitoring these metrics by choosing one\-minute or five\-minute granularity\. 

Instance monitoring is enabled whenever an instance is launched, using either basic monitoring \(five\-minute granularity\) or detailed monitoring \(one\-minute granularity\)\. For detailed monitoring, additional charges apply\. For more information, see [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) and [Monitoring your instances using CloudWatch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch.html) in the *Amazon EC2 User Guide for Linux Instances*\.

Before creating an Auto Scaling group, you should create a launch template or launch configuration that permits the type of monitoring that is appropriate to your application\. If you add a scaling policy to your group, we strongly recommend that you use detailed monitoring to get metric data for EC2 instances at a one\-minute granularity, because that achieves a faster response to changes in load\.

**Topics**
+ [Enable detailed monitoring \(console\)](#enable-detailed-monitoring-console)
+ [Enable detailed monitoring \(AWS CLI\)](#enable-detailed-monitoring-cli)
+ [Switch between basic and detailed monitoring](#change-monitoring)
+ [Collect additional metrics using the CloudWatch agent](#metrics-collected-by-cloudwatch-agent)

## Enable detailed monitoring \(console\)<a name="enable-detailed-monitoring-console"></a>

By default, basic monitoring is enabled when you use the AWS Management Console to create a launch template or launch configuration\. 

**To enable detailed monitoring in a launch template**  
When you create the launch template using the AWS Management Console, in the **Advanced details** section, for **Detailed CloudWatch monitoring**, choose **Enable**\. Otherwise, basic monitoring is enabled\. For more information, see [Configure advanced settings for your launch template](create-launch-template.md#advanced-settings-for-your-launch-template)\.

**To enable detailed monitoring in a launch configuration**  
When you create the launch configuration using the AWS Management Console, in the **Additional configuration** section, select **Enable EC2 instance detailed monitoring within CloudWatch**\. Otherwise, basic monitoring is enabled\. For more information, see [Create a launch configuration](create-launch-config.md)\.

## Enable detailed monitoring \(AWS CLI\)<a name="enable-detailed-monitoring-cli"></a>

By default, basic monitoring is enabled when you create a launch template using the AWS CLI\. Detailed monitoring is enabled by default when you create a launch configuration using the AWS CLI\. 

**To enable detailed monitoring in a launch template**  
For launch templates, use the [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command and pass a JSON file that contains the information for creating the launch template\. Set the monitoring attribute to `"Monitoring":{"Enabled":true}` to enable detailed monitoring or `"Monitoring":{"Enabled":false}` to enable basic monitoring\. 

**To enable detailed monitoring in a launch configuration**  
For launch configurations, use the [create\-launch\-configuration](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command with the `--instance-monitoring` option\. Set this option to `true` to enable detailed monitoring or `false` to enable basic monitoring\.

```
--instance-monitoring Enabled=true
```

## Switch between basic and detailed monitoring<a name="change-monitoring"></a>

To change the type of monitoring enabled on new EC2 instances, update the launch template or update the Auto Scaling group to use a new launch template or launch configuration\. Existing instances continue to use the previously enabled monitoring type\. To update all instances, terminate them so that they are replaced by your Auto Scaling group or update instances individually using [monitor\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/monitor-instances.html) and [unmonitor\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/unmonitor-instances.html)\.

**Note**  
With the instance refresh and maximum instance lifetime features, you can also replace all instances in the Auto Scaling group to launch new instances that use the new settings\. For more information, see [Replace Auto Scaling instances](ec2-auto-scaling-group-replacing-instances.md)\.

When you switch between basic and detailed monitoring:

If you have CloudWatch alarms associated with your Auto Scaling group, use the [put\-metric\-alarm](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to update each alarm\. Make each period match the monitoring type \(300 seconds for basic monitoring and 60 seconds for detailed monitoring\)\. If you change from detailed monitoring to basic monitoring but do not update your alarms to match the five\-minute period, they continue to check for statistics every minute\. They might find no data available for as many as four out of every five periods\.

## Collect additional metrics using the CloudWatch agent<a name="metrics-collected-by-cloudwatch-agent"></a>

To collect operating system\-level metrics like available and used memory, you must install the CloudWatch agent\. Additional fees may apply\. For more information, see [Metrics collected by the CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html) in the *Amazon CloudWatch User Guide*\.