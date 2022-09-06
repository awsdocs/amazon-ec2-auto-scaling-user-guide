# View monitoring graphs in the Amazon EC2 Auto Scaling console<a name="viewing-monitoring-graphs"></a>

In the Amazon EC2 Auto Scaling section of the Amazon EC2 console, you can monitor minute\-by\-minute progress of individual Auto Scaling groups using CloudWatch metrics\. 

You can monitor the following types of metrics: 
+ **Auto Scaling metrics** – Auto Scaling metrics are turned on only when you enable them\. For more information, see [Enable Auto Scaling group metrics \(console\)](ec2-auto-scaling-cloudwatch-monitoring.md#as-enable-group-metrics)\. When Auto Scaling metrics are enabled, the monitoring graphs show data published at one\-minute granularity for Auto Scaling metrics\.
+ **EC2 metrics** – If detailed monitoring is enabled, the monitoring graphs show data published at one\-minute granularity for instance metrics\. For more information, see [Configure monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\. 

**To view monitoring graphs using the Amazon EC2 Auto Scaling console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to the Auto Scaling group that you want to view metrics for\. 

   A split pane opens up in the bottom part of the **Auto Scaling groups** page\.

1. Choose the **Monitoring** tab\.

   Amazon EC2 Auto Scaling displays monitoring graphs for **Auto Scaling** metrics\.

1. To view monitoring graphs of the aggregated instance metrics for the group, choose **EC2**\. 

 **Graph actions** 
+ Hover on a data point to view a data pop\-up for a specific time in UTC\. 
+ To enlarge a graph, choose **Enlarge** from the menu tool \(the three vertical dots\) in the upper right of the graph\. Alternatively, choose the maximize icon at the top of the graph\.
+ Adjust the time period for data displayed in the graph by selecting one of the predefined time period values\. If the graph is enlarged, you can choose **Custom** to define your own time period\.
+ Choose **Refresh** from the menu tool to update the data in a graph\.
+ Drag your cursor over the graph data to select a specific range\. You can then choose **Apply time range** in the menu tool\.
+ Choose **View logs** from the menu tool to view associated log streams \(if any\) in the CloudWatch console\.
+ To view a graph in CloudWatch, choose **View in metrics** from the menu tool\. This takes you to the CloudWatch page for that graph\. There, you can view more information or access historical information to gain a better understanding of how your Auto Scaling group changed over an extended period\. 

## Graph metrics for your Auto Scaling groups<a name="graph-metrics"></a>

After you create an Auto Scaling group, you can open the Amazon EC2 Auto Scaling console and view the monitoring graphs for the group on the **Monitoring** tab\. 

In the **Auto Scaling** section, the graph metrics include the following overall metrics\. These metrics provide measurements that can be indicators of a potential issue, such as number of terminating instances or number of pending instances\.
+ **Minimum Group Size** \(based on `GroupMinSize`\)
+ **Maximum Group Size** \(based on `GroupMaxSize`\)
+ **Desired Capacity** \(based on `GroupDesiredCapacity`\)
+ **In Service Instances** \(based on `GroupInServiceInstances`\)
+ **Pending Instances** \(based on `GroupPendingInstances`\)
+ **Standby Instances** \(based on `GroupStandbyInstances`\)
+ **Terminating Instances** \(based on `GroupTerminatingInstances`\)
+ **Total Instances** \(based on `GroupTotalInstances`\)

In the **EC2** section, you can find the following graph metrics based on key performance metrics for your Amazon EC2 instances\. These EC2 metrics are an aggregate of metrics for all instances in the group\.
+ **CPU Utilization** \(based on `CPUUtilization`\)
+ **Disk Reads** \(based on `DiskReadBytes`\)
+ **Disk Read Operations** \(based on `DiskReadOps`\)
+ **Disk Writes** \(based on `DiskWriteBytes`\)
+ **Disk Write Operations** \(based on `DiskWriteOps`\)
+ **Network In** \(based on `NetworkIn`\)
+ **Network Out** \(based on `NetworkOut`\)
+ **Status Check Failed \(Any\)** \(based on `StatusCheckFailed`\)
+ **Status Check Failed \(Instance\)** \(based on `StatusCheckFailed_Instance`\)
+ **Status Check Failed \(System\)** \(based on `StatusCheckFailed_System`\)

You can use the next set of **Auto Scaling** graph metrics to measure specific features of specific groups\.

The following metric data is available for groups where instances have weights that define how many units each instance contributes to the desired capacity of the group:
+ **In Service Capacity Units** \(based on `GroupInServiceCapacity`\)
+ **Pending Capacity Units** \(based on `GroupPendingCapacity`\)
+ **Standby Capacity Units** \(based on `GroupStandbyCapacity`\)
+ **Terminating Capacity Units** \(based on `GroupTerminatingCapacity`\)
+ **Total Capacity Units** \(based on `GroupTotalCapacity`\)

The following metric data is available if the group uses the [warm pool](ec2-auto-scaling-warm-pools.md) feature:
+ **Warm Pool Minimum Size** \(based on `WarmPoolMinSize`\)
+ **Warm Pool Desired Capacity** \(based on `WarmPoolDesiredCapacity`\)
+ **Warm Pool Pending Capacity Units** \(based on `WarmPoolPendingCapacity`\)
+ **Warm Pool Terminating Capacity Units** \(based on `WarmPoolTerminatingCapacity`\)
+ **Warm Pool Warmed Capacity Units** \(based on `WarmPoolWarmedCapacity`\)
+ **Warm Pool Total Capacity Units Launched** \(based on `WarmPoolTotalCapacity`\)
+ **Group and Warm Pool Desired Capacity** \(based on `GroupAndWarmPoolDesiredCapacity`\)
+ **Group and Warm Pool Total Capacity Units Launched** \(based on `GroupAndWarmPoolTotalCapacity`\)

### See also<a name="graph-metrics-see-also"></a>

For information about the data source for the **EC2** graph metrics, see [List the available CloudWatch metrics for your instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

To monitor per\-instance metrics, see [Graph metrics for your instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/graphs-in-the-aws-management-console.html) in the *Amazon EC2 User Guide for Linux Instances*\. 