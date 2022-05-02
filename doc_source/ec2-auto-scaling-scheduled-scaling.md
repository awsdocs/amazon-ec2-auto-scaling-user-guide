# Scheduled scaling for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-scheduled-scaling"></a>

Scheduled scaling helps you to set up your own scaling schedule according to predictable load changes\. For example, let's say that every week the traffic to your web application starts to increase on Wednesday, remains high on Thursday, and starts to decrease on Friday\. You can configure a schedule for Amazon EC2 Auto Scaling to increase capacity on Wednesday and decrease capacity on Friday\. 

To use scheduled scaling, you create *scheduled actions*\. Scheduled actions are performed automatically as a function of date and time\. When you create a scheduled action, you specify when the scaling activity should occur and the new desired, minimum, and maximum sizes for the scaling action\. You can create scheduled actions that scale one time only or that scale on a recurring schedule\. 

**Topics**
+ [Considerations](#sch-actions_rules)
+ [Recurring schedules](#sch-actions_recurring_schedules)
+ [Create and manage scheduled actions \(console\)](#create-sch-actions)
+ [Create and manage scheduled actions \(AWS CLI\)](#create-sch-actions-aws-cli)
+ [Limitations](#scheduled-scaling-limitations)

## Considerations<a name="sch-actions_rules"></a>

When you create a scheduled action, keep the following in mind:
+ A scheduled action sets the desired, minimum, and maximum sizes to what is specified by the scheduled action at the date and time specified\. The request can optionally include only one of these sizes\. For example, you can create a scheduled action with only the desired capacity specified\. In some cases, however, you must include the minimum and maximum sizes to ensure that the new desired capacity that you specified in the action is not outside of these limits\.
+ By default, the recurring schedules that you set are in Coordinated Universal Time \(UTC\)\. You can change the time zone to correspond to your local time zone or a time zone for another part of your network\. When you specify a time zone that observes Daylight Saving Time \(DST\), the action automatically adjusts for DST\. 
+ You can temporarily turn off scheduled scaling for an Auto Scaling group by suspending the `ScheduledActions` process\. This helps you prevent scheduled actions from being active without having to delete them\. You can then resume scheduled scaling when you want to use it again\. For more information, see [Suspend and resume a process for an Auto Scaling group](as-suspend-resume-processes.md)\.
+ The order of execution for scheduled actions is guaranteed within the same group, but not for scheduled actions across groups\.
+ A scheduled action generally executes within seconds\. However, the action might be delayed for up to two minutes from the scheduled start time\. Because scheduled actions within an Auto Scaling group are executed in the order that they are specified, actions with scheduled start times close to each other can take longer to execute\.

## Recurring schedules<a name="sch-actions_recurring_schedules"></a>

You can create scheduled actions that scale your Auto Scaling group on a recurring schedule\. 

To create a recurring schedule using the AWS CLI or an SDK, specify a cron expression and a time zone to describe when that schedule action is to recur\. You can optionally specify a date and time for the start time, the end time, or both\. 

To create a recurring schedule using the AWS Management Console, specify the recurrence pattern, time zone, start time, and optional end time of your scheduled action\. All of the recurrence pattern options are based on cron expressions\. Alternatively, you can write your own custom cron expression\. 

The supported cron expression format consists of five fields separated by white spaces: \[Minute\] \[Hour\] \[Day\_of\_Month\] \[Month\_of\_Year\] \[Day\_of\_Week\]\. For example, the cron expression `30 6 * * 2` configures a scheduled action that recurs every Tuesday at 6:30 AM\. The asterisk is used as a wildcard to match all values for a field\. For other examples of cron expressions, see [https://crontab\.guru/examples\.html](https://crontab.guru/examples.html)\. For information about writing your own cron expressions in this format, see [Crontab](http://crontab.org)\. 

Choose your start and end times carefully\. Keep the following in mind:
+ If you specify a start time, Amazon EC2 Auto Scaling performs the action at this time, and then performs the action based on the specified recurrence\.
+ If you specify an end time, the action stops repeating after this time\. A scheduled action does not persist in your account once it has reached its end time\.
+ The start time and end time must be set in UTC when you use the AWS CLI or an SDK\.

## Create and manage scheduled actions \(console\)<a name="create-sch-actions"></a>

Use the procedures in this section to create and manage scheduled actions using the AWS Management Console\.

If you create a scheduled action using the console and specify a time zone that observes Daylight Saving Time \(DST\), both the recurring schedule and the start and end times automatically adjust for DST\. 

### Create a scheduled action<a name="create-sch-action"></a>

Complete the following procedure to create a scheduled action to scale your Auto Scaling group\.

**To create a scheduled action for an Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\.

1. On the **Automatic scaling** tab, in **Scheduled actions**, choose **Create scheduled action**\.

1. Enter a **Name** for the scheduled action\.

1. For **Desired capacity**, **Min**, **Max**, choose the new desired size of the group and the new minimum and maximum capacity\.

1. For **Recurrence**, choose one of the available options\.
   + If you want to scale on a recurring schedule, choose how often Amazon EC2 Auto Scaling should run the scheduled action\. 
     + If you choose an option that begins with **Every**, the cron expression is created for you\.
     + If you choose **Cron**, enter a cron expression that specifies when to perform the action\. 
   + If you want to scale only once, choose **Once**\.

1. For **Time zone**, choose a time zone\. The default is `Etc/UTC`\.
**Note**  
All of the time zones listed are from the IANA Time Zone database\. For more information, see [https://en\.wikipedia\.org/wiki/List\_of\_tz\_database\_time\_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)\.

1. Define a date and time for **Specific start time**\.
   + If you chose a recurring schedule, the start time defines when the first scheduled action in the recurring series runs\. 
   + If you chose **Once** as the recurrence, the start time defines the date and time for the schedule action to run\. 

1.  \(Optional\) For recurring schedules, you can specify an end time by choosing **Set End Time** and then choosing a date and time for **End by**\.

1. Choose **Create**\. The console displays the scheduled actions for the Auto Scaling group\. 

### Verify the time, date, and time zone<a name="verify-sch-action"></a>

To verify whether your time, date, and time zone are configured correctly, check the **Start time**, **End time**, and **Time zone** values in the **Scheduled actions** table on the **Automatic scaling** tab for your Auto Scaling group\. 

Amazon EC2 Auto Scaling shows the values for **Start time** and **End time** in your local time with the UTC offset in effect at the specified date and time\. The UTC offset is the difference, in hours and minutes, from local time to UTC\. The value for **Time zone** shows your requested time zone, for example, `America/New_York`\. 

Location\-based time zones such as `America/New_York` automatically adjust for Daylight Savings Time \(DST\)\. However, a UTC\-based time zone such as `Etc/UTC` is an absolute time and will not adjust for DST\. 

For example, you have a recurring schedule whose time zone is `America/New_York`\. The first scaling action happens in the `America/New_York` time zone before DST starts\. The next scaling action happens in the `America/New_York` time zone after DST starts\. The first action starts at 8:00 AM UTC\-5 in local time, while the second time starts at 8:00 AM UTC\-4 in local time\.

### Update a scheduled action<a name="update-sch-action"></a>

After creating a scheduled action, you can update any of its settings except the name\.

**To update a scheduled action**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\.

1. On the **Automatic scaling** tab, in **Scheduled actions**, select a scheduled action\.

1. Choose **Actions**, **Edit**\.

1. Make the needed changes and choose **Save changes**\.

### Delete a scheduled action<a name="delete-sch-action"></a>

When you no longer need a scheduled action, you can delete it\.

**To delete a scheduled action**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select your Auto Scaling group\.

1. On the **Automatic scaling** tab, in **Scheduled actions**, select a scheduled action\.

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

## Create and manage scheduled actions \(AWS CLI\)<a name="create-sch-actions-aws-cli"></a>

You can create and update scheduled actions that scale one time only or that scale on a recurring schedule using the [put\-scheduled\-update\-group\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command\. 

### Create a scheduled action that occurs only once<a name="one-time-schedule"></a>

To automatically scale your Auto Scaling group one time only, at a specified date and time, use the `--start-time "YYYY-MM-DDThh:mm:ssZ"` option\.

**Example: To scale out one time only**  
To increase the number of running instances in your Auto Scaling group at a specific time, use the following command\. 

At the date and time specified for `--start-time` \(8:00 AM UTC on March 31, 2021\), if the group currently has fewer than 3 instances, it scales out to 3 instances\. 

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-one-time-action \
  --auto-scaling-group-name my-asg --start-time "2021-03-31T08:00:00Z" --desired-capacity 3
```

**Example: To scale in one time only**  
To decrease the number of running instances in your Auto Scaling group at a specific time, use the following command\. 

At the date and time specified for `--start-time` \(4:00 PM UTC on March 31, 2021\), if the group currently has more than 1 instance, it scales in to 1 instance\. 

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-one-time-action \
  --auto-scaling-group-name my-asg --start-time "2021-03-31T16:00:00Z" --desired-capacity 1
```

### Create a scheduled action that runs on a recurring schedule<a name="recurrence-schedule-cron"></a>

To schedule scaling on a recurring schedule, use the `--recurrence "cron expression"` option\. 

The following is an example of a scheduled action that specifies a cron expression\. 

On the specified schedule \(every day at 9:00 AM UTC\), if the group currently has fewer than 3 instances, it scales out to 3 instances\. If the group currently has more than 3 instances, it scales in to 3 instances\.

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-recurring-action \
  --auto-scaling-group-name my-asg --recurrence "0 9 * * *" --desired-capacity 3
```

### Create a recurring scheduled action that specifies a time zone<a name="recurring-schedule-set-time-zone"></a>

Scheduled actions are set to the UTC time zone by default\. To specify a different time zone, include the `--time-zone` option and specify the canonical name for the IANA time zone \(`America/New_York`, for example\)\. For more information, see [https://en\.wikipedia\.org/wiki/List\_of\_tz\_database\_time\_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)\.

The following is an example that uses the `--time-zone` option when creating a recurring scheduled action to scale capacity\.

On the specified schedule \(every Monday through Friday at 6:00 PM local time\), if the group currently has fewer than 2 instances, it scales out to 2 instances\. If the group currently has more than 2 instances, it scales in to 2 instances\.

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-recurring-action \
  --auto-scaling-group-name my-asg --recurrence "0 18 * * 1-5" --time-zone "America/New_York" \
  --desired-capacity 2
```

### Describe scheduled actions<a name="describe-scheduled-actions"></a>

To describe the scheduled actions for an Auto Scaling group, use the following [describe\-scheduled\-actions](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scheduled-actions.html) command\.

```
aws autoscaling describe-scheduled-actions --auto-scaling-group-name my-asg
```

If successful, this command returns output similar to the following\.

```
{
  "ScheduledUpdateGroupActions": [
    {
      "AutoScalingGroupName": "my-asg",
      "ScheduledActionName": "my-recurring-action",
      "Recurrence": "30 0 1 1,6,12 *",
      "ScheduledActionARN": "arn:aws:autoscaling:us-west-2:123456789012:scheduledUpdateGroupAction:8e86b655-b2e6-4410-8f29-b4f094d6871c:autoScalingGroupName/my-asg:scheduledActionName/my-recurring-action",
      "StartTime": "2020-12-01T00:30:00Z",
      "Time": "2020-12-01T00:30:00Z",
      "MinSize": 1,
      "MaxSize": 6,
      "DesiredCapacity": 4
    }
  ]
}
```

### Delete a scheduled action<a name="delete-sch-actions-aws-cli"></a>

To delete a scheduled action, use the following [delete\-scheduled\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-scheduled-action.html) command\.

```
aws autoscaling delete-scheduled-action --auto-scaling-group-name my-asg \
  --scheduled-action-name my-recurring-action
```

## Limitations<a name="scheduled-scaling-limitations"></a>
+ The names of scheduled actions must be unique per Auto Scaling group\. 
+ A scheduled action must have a unique time value\. If you attempt to schedule an activity at a time when another scaling activity is already scheduled, the call is rejected and returns an error indicating that a scheduled action with this scheduled start time already exists\.
+ You can create a maximum of 125 scheduled actions per Auto Scaling group\.