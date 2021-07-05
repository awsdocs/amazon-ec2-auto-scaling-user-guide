# Exploring your data and forecast<a name="predictive-scaling-graphs"></a>

After the forecast is created, you can view charts showing historical data from the last eight weeks and the forecast for the next two days\. To find detailed information about a forecast and its history, open the Auto Scaling group from the Amazon EC2 Auto Scaling console and choose the **Automatic scaling** tab in the lower pane\. The graphs become available shortly after the policy is created\.

Each graph displays forecast values against actual values and a particular type of data\. The **Load** graph shows load forecast and actual values for the load metric you choose\. The **Capacity** graph shows the number of instances that are forecasted based on your target utilization and the actual number of instances launched\. Various colors show actual metric data points and past and future forecast values\. The orange line shows the actual metric data points\. The green line shows the forecast that was generated for the future forecast period\. The blue line shows the forecast for past periods\. 

![\[The graphs for a predictive scaling policy\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/asg-console-predictive-scaling-policy-graphs.png)

You can adjust the time range for past data by choosing your preferred value in the top right of the graph: **2 days**, **1 week**, **2 weeks**, **4 weeks**, **6 weeks**, or **8 weeks**\. Each point on the graph represents one hour of data\. When hovering over a data point, the tooltip shows the value for a particular point in time, in UTC\.

To enlarge the graph pane, choose the expand icon in the top right of the graph\. To revert back to the default view, choose the icon again\.

You can also use the AWS CLI command get\-predictive\-scaling\-forecast to get forecast data\. The data returned by this call can help you identify time periods when you might want to override the forecast\. For more information, see [Overriding forecast values using scheduled actions](predictive-scaling-overriding-forecast-capacity.md)\.

**Note**  
We recommend that you enable Auto Scaling group metrics\. If these metrics are not enabled, actual capacity data will be missing from the capacity forecast graph\. There is no cost for enabling these metrics\.   
To enable Auto Scaling group metrics, open the Auto Scaling group in the Amazon EC2 console, and from the **Monitoring** tab, select the **Auto Scaling group metrics collection**, **Enable** check box\. For more information, see [Enable Auto Scaling group metrics \(console\)](as-instance-monitoring.md#as-enable-group-metrics)\.

**Important**  
If the Auto Scaling group is new, allow 24 hours for Amazon EC2 Auto Scaling to create the first forecast\. 