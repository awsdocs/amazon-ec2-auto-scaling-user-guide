# Override forecast values using scheduled actions<a name="predictive-scaling-overriding-forecast-capacity"></a>

Sometimes, you might have additional information about your future application requirements that the forecast calculation is unable to take into account\. For example, forecast calculations might underestimate the capacity needed for an upcoming marketing event\. You can use scheduled actions to temporarily override the forecast during future time periods\. The scheduled actions can run on a recurring basis, or at a specific date and time when there are one\-time demand fluctuations\. 

For example, you can create a scheduled action with a higher minimum capacity than what is forecasted\. At runtime, Amazon EC2 Auto Scaling updates the minimum capacity of your Auto Scaling group\. Because predictive scaling optimizes for capacity, a scheduled action with a minimum capacity that is higher than the forecast values is honored\. This prevents capacity from being less than expected\. To stop overriding the forecast, use a second scheduled action to return the minimum capacity to its original setting\.

The following procedure outlines the steps for overriding the forecast during future time periods\. 

**Topics**
+ [Step 1: \(Optional\) Analyze time series data](#analyzing-time-series-data)
+ [Step 2: Create two scheduled actions](#scheduling-capacity)

## Step 1: \(Optional\) Analyze time series data<a name="analyzing-time-series-data"></a>

Start by analyzing the forecast time series data\. This is an optional step, but it is helpful if you want to understand the details of the forecast\.

1. **Retrieve the forecast**

   After the forecast is created, you can query for a specific time period in the forecast\. The goal of the query is to get a complete view of the time series data for a specific time period\. 

   Your query can include up to two days of future forecast data\. If you have been using predictive scaling for a while, you can also access your past forecast data\. However, the maximum time duration between the start and end time is 30 days\. 

   To get the forecast using the [get\-predictive\-scaling\-forecast](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/get-predictive-scaling-forecast.html) AWS CLI command, provide the following parameters in the command: 
   + Enter the name of the Auto Scaling group in the `--auto-scaling-group-name` parameter\. 
   + Enter the name of the policy in the `--policy-name` parameter\. 
   + Enter the start time in the `--start-time` parameter to return only forecast data for after or at the specified time\.
   + Enter the end time in the `--end-time` parameter to return only forecast data for before the specified time\. 

   ```
   aws autoscaling get-predictive-scaling-forecast --auto-scaling-group-name my-asg \
       --policy-name cpu40-predictive-scaling-policy \
       --start-time "2021-05-19T17:00:00Z" \
       --end-time "2021-05-19T23:00:00Z"
   ```

   If successful, the command returns data similar to the following example\. 

   ```
   {
       "LoadForecast": [
           {
               "Timestamps": [
                   "2021-05-19T17:00:00+00:00",
                   "2021-05-19T18:00:00+00:00",
                   "2021-05-19T19:00:00+00:00",
                   "2021-05-19T20:00:00+00:00",
                   "2021-05-19T21:00:00+00:00",
                   "2021-05-19T22:00:00+00:00",
                   "2021-05-19T23:00:00+00:00"
               ],
               "Values": [
                   153.0655799339254,
                   128.8288551285919,
                   107.1179447150675,
                   197.3601844551528,
                   626.4039934516954,
                   596.9441277518481,
                   677.9675713779869
               ],
               "MetricSpecification": {
                   "TargetValue": 40.0,
                   "PredefinedMetricPairSpecification": {
                       "PredefinedMetricType": "ASGCPUUtilization"
                   }
               }
           }
       ],
       "CapacityForecast": {
           "Timestamps": [
               "2021-05-19T17:00:00+00:00",
               "2021-05-19T18:00:00+00:00",
               "2021-05-19T19:00:00+00:00",
               "2021-05-19T20:00:00+00:00",
               "2021-05-19T21:00:00+00:00",
               "2021-05-19T22:00:00+00:00",
               "2021-05-19T23:00:00+00:00"
           ],
           "Values": [
               2.0,
               2.0,
               2.0,
               2.0,
               4.0,
               4.0,
               4.0
           ]
       },
       "UpdateTime": "2021-05-19T01:52:50.118000+00:00"
   }
   ```

   The response includes two forecasts: `LoadForecast` and `CapacityForecast`\. `LoadForecast` shows the hourly load forecast\. `CapacityForecast` shows forecast values for the capacity that is needed on an hourly basis to handle the forecasted load while maintaining a `TargetValue` of 40\.0 \(40% average CPU utilization\)\.

1. **Identify the target time period**

   Identify the hour or hours when the one\-time demand fluctuation should take place\. Remember that dates and times shown in the forecast are in UTC\.

## Step 2: Create two scheduled actions<a name="scheduling-capacity"></a>

Next, create two scheduled actions for a specific time period when your application will have a higher than forecasted load\. For example, if you have a marketing event that will drive traffic to your site for a limited period of time, you can schedule a one\-time action to update the minimum capacity when it starts\. Then, schedule another action to return the minimum capacity to the original setting when the event ends\. 

**To create two scheduled actions for one\-time events \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Automatic scaling** tab, in **Scheduled actions**, choose **Create scheduled action**\.

1. Fill in the following scheduled action settings:

   1. Enter a **Name** for the scheduled action\.

   1. For **Min**, enter the new minimum capacity for your Auto Scaling group\. The **Min** must be less than or equal to the maximum size of the group\. If your value for **Min** is greater than group's maximum size, you must update **Max**\. 

   1. For **Recurrence**, choose **Once**\.

   1. For **Time zone**, choose a time zone\. If no time zone is chosen, `ETC/UTC` is used by default\.

   1. Define a **Specific start time**\. 

1. Choose **Create**\.

   The console displays the scheduled actions for the Auto Scaling group\. 

1. Configure a second scheduled action to return the minimum capacity to the original setting at the end of the event\. Predictive scaling can scale capacity only when the value you set for **Min** is lower than the forecast values\.

**To create two scheduled actions for one\-time events \(AWS CLI\)**  
To use the AWS CLI to create the scheduled actions, use the [put\-scheduled\-update\-group\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command\. 

For example, let's define a schedule that maintains a minimum capacity of three instances on May 19 at 5:00 PM for eight hours\. The following commands show how to implement this scenario\.

The first [put\-scheduled\-update\-group\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command instructs Amazon EC2 Auto Scaling to update the minimum capacity of the specified Auto Scaling group at 5:00 PM UTC on May 19, 2021\. 

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-event-start \
  --auto-scaling-group-name my-asg --start-time "2021-05-19T17:00:00Z" --minimum-capacity 3
```

The second command instructs Amazon EC2 Auto Scaling to set the group's minimum capacity to one at 1:00 AM UTC on May 20, 2021\. 

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-event-end \
  --auto-scaling-group-name my-asg --start-time "2021-05-20T01:00:00Z" --minimum-capacity 1
```

After you add these scheduled actions to the Auto Scaling group, Amazon EC2 Auto Scaling does the following: 
+ At 5:00 PM UTC on May 19, 2021, the first scheduled action runs\. If the group currently has fewer than three instances, the group scales out to three instances\. During this time and for the next eight hours, Amazon EC2 Auto Scaling can continue to scale out if the predicted capacity is higher than the actual capacity or if there is a dynamic scaling policy in effect\. 
+ At 1:00 AM UTC on May 20, 2021, the second scheduled action runs\. This returns the minimum capacity to its original setting at the end of the event\.

### Scaling based on recurring schedules<a name="scheduling-recurring-actions"></a>

To override the forecast for the same time period every week, create two scheduled actions and provide the time and date logic using a cron expression\. 

The cron expression format consists of five fields separated by spaces: \[Minute\] \[Hour\] \[Day\_of\_Month\] \[Month\_of\_Year\] \[Day\_of\_Week\]\. Fields can contain any allowed values, including special characters\. 

For example, the following cron expression runs the action every Tuesday at 6:30 AM\. The asterisk is used as a wildcard to match all values for a field\.

```
30 6 * * 2
```

### See also<a name="scheduling-scaling-see-also"></a>

For more information about how to create, list, edit, and delete scheduled actions, see [Scheduled scaling for Amazon EC2 Auto Scaling](ec2-auto-scaling-scheduled-scaling.md)\.