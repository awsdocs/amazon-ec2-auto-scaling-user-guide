# Configuring monitoring for Auto Scaling instances<a name="enable-as-instance-metrics"></a>

Amazon EC2 can enable detailed monitoring when it is launching EC2 instances in your Auto Scaling group\. You configure monitoring for Auto Scaling instances using a launch template or launch configuration\. 

Monitoring is enabled whenever an instance is launched, either basic monitoring \(5\-minute granularity\) or detailed monitoring \(1\-minute granularity\)\. For detailed monitoring, additional charges apply\. For more information, see [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) and [Monitoring your instances using CloudWatch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch.html) in the *Amazon EC2 User Guide for Linux Instances*\.

By default, basic monitoring is enabled when you create a launch template or when you use the AWS Management Console to create a launch configuration\. Detailed monitoring is enabled by default when you create a launch configuration using the AWS CLI or an SDK\. 

To change the type of monitoring enabled on new EC2 instances, update the launch template or update the Auto Scaling group to use a new launch configuration\. Existing instances continue to use the previously enabled monitoring type\. To update all instances, terminate them so that they are replaced by your Auto Scaling group or update instances individually using [https://docs.aws.amazon.com/cli/latest/reference/ec2/monitor-instances.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/monitor-instances.html) and [https://docs.aws.amazon.com/cli/latest/reference/ec2/unmonitor-instances.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/unmonitor-instances.html)\.

**Note**  
With the maximum instance lifetime and instance refresh features, you can also replace all instances in the Auto Scaling group to launch new instances that use the new settings\. For more information, see [Replacing Auto Scaling instances based on maximum instance lifetime](asg-max-instance-lifetime.md) and [Replacing Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

If you have CloudWatch alarms associated with your Auto Scaling group, use the [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html) command to update each alarm\. Make each period match the monitoring type \(300 seconds for basic monitoring and 60 seconds for detailed monitoring\)\. If you change from detailed monitoring to basic monitoring but do not update your alarms to match the five\-minute period, they continue to check for statistics every minute\. They might find no data available for as many as four out of every five periods\.

**To configure CloudWatch monitoring \(console\)**  
When you create the launch configuration using the AWS Management Console, on the **Configure Details** page, select **Enable CloudWatch detailed monitoring**\. Otherwise, basic monitoring is enabled\. For more information, see [Creating a launch configuration](create-launch-config.md)\.

To enable detailed monitoring for a launch template using the AWS Management Console, in the **Advanced Details** section, for **Monitoring**, choose **Enable**\. Otherwise, basic monitoring is enabled\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.

**To configure CloudWatch monitoring \(AWS CLI\)**  
For launch configurations, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command with the `--instance-monitoring` option\. Set this option to `true` to enable detailed monitoring or `false` to enable basic monitoring\.

```
--instance-monitoring Enabled=true
```

For launch templates, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command and pass a JSON file that contains the information for creating the launch template\. Set the monitoring attribute to `"Monitoring":{"Enabled":true}` to enable detailed monitoring or `"Monitoring":{"Enabled":false}` to enable basic monitoring\. 