# Configuring monitoring for Auto Scaling instances<a name="enable-as-instance-metrics"></a>

Amazon EC2 can enable detailed monitoring when it is launching EC2 instances in your Auto Scaling group\. You configure monitoring for Auto Scaling instances using a launch template or launch configuration\. 

Monitoring is enabled whenever an instance is launched, either basic monitoring \(5\-minute granularity\) or detailed monitoring \(1\-minute granularity\)\. For detailed monitoring, additional charges apply\. For more information, see [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) and [Monitoring your instances using CloudWatch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch.html) in the *Amazon EC2 User Guide for Linux Instances*\.

## Enable detailed monitoring \(console\)<a name="enable-detailed-monitoring-console"></a>

By default, basic monitoring is enabled when you use the AWS Management Console to create a launch template or launch configuration\. 

**To enable detailed monitoring in a launch template**  
When you create the launch template using the AWS Management Console, in the **Advanced details** section, for **Detailed CloudWatch monitoring**, choose **Enable**\. Otherwise, basic monitoring is enabled\. For more information, see [Configuring advanced settings for your launch template](create-launch-template.md#advanced-settings-for-your-launch-template)\.

**To enable detailed monitoring in a launch configuration**  
When you create the launch configuration using the AWS Management Console, in the **Additional configuration** section, select **Enable EC2 instance detailed monitoring within CloudWatch**\. Otherwise, basic monitoring is enabled\. For more information, see [Creating a launch configuration](create-launch-config.md)\.

## Enable detailed monitoring \(AWS CLI\)<a name="enable-detailed-monitoring-cli"></a>

By default, basic monitoring is enabled when you create a launch template using the AWS CLI\. Detailed monitoring is enabled by default when you create a launch configuration using the AWS CLI\. 

**To enable detailed monitoring in a launch template**  
For launch templates, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command and pass a JSON file that contains the information for creating the launch template\. Set the monitoring attribute to `"Monitoring":{"Enabled":true}` to enable detailed monitoring or `"Monitoring":{"Enabled":false}` to enable basic monitoring\. 

**To enable detailed monitoring in a launch configuration**  
For launch configurations, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command with the `--instance-monitoring` option\. Set this option to `true` to enable detailed monitoring or `false` to enable basic monitoring\.

```
--instance-monitoring Enabled=true
```

## Switching between basic and detailed monitoring<a name="change-monitoring"></a>

To change the type of monitoring enabled on new EC2 instances, update the launch template or update the Auto Scaling group to use a new launch template or launch configuration\. Existing instances continue to use the previously enabled monitoring type\. To update all instances, terminate them so that they are replaced by your Auto Scaling group or update instances individually using [https://docs.aws.amazon.com/cli/latest/reference/ec2/monitor-instances.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/monitor-instances.html) and [https://docs.aws.amazon.com/cli/latest/reference/ec2/unmonitor-instances.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/unmonitor-instances.html)\.

**Note**  
With the instance refresh and maximum instance lifetime features, you can also replace all instances in the Auto Scaling group to launch new instances that use the new settings\. For more information, see [Replacing Auto Scaling instances](ec2-auto-scaling-group-replacing-instances.md)\.

When you switch between basic and detailed monitoring:

If you have CloudWatch alarms associated with your Auto Scaling group, use the [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to update each alarm\. Make each period match the monitoring type \(300 seconds for basic monitoring and 60 seconds for detailed monitoring\)\. If you change from detailed monitoring to basic monitoring but do not update your alarms to match the five\-minute period, they continue to check for statistics every minute\. They might find no data available for as many as four out of every five periods\.