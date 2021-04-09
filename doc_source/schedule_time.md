# Scheduled scaling for Amazon EC2 Auto Scaling<a name="schedule_time"></a>

Scheduled scaling helps you to set up your own scaling schedule according to predictable load changes\. For example, let's say that every week the traffic to your web application starts to increase on Wednesday, remains high on Thursday, and starts to decrease on Friday\. You can configure a schedule for Amazon EC2 Auto Scaling to increase capacity on Wednesday and decrease capacity on Friday\. 

To use scheduled scaling, you create *scheduled actions*\. Scheduled actions are performed automatically as a function of date and time\. When you create a scheduled action, you specify when the scaling activity should occur and the new desired, minimum, and maximum sizes for the scaling action\. You can create scheduled actions that scale one time only or that scale on a recurring schedule\.

**Contents**
+ [Considerations](#sch-actions_rules)
+ [Create and manage scheduled actions \(console\)](#create-sch-actions)
  + [Update a scheduled action](#update-sch-action)
  + [Delete a scheduled action](#delete-sch-action)
+ [Create and manage scheduled actions \(AWS CLI\)](#create-sch-actions-aws-cli)
  + [Create a scheduled action that occurs only once](#one-time-schedule)
  + [Create a scheduled action that runs on a recurring schedule](#recurrence-schedule-cron)
  + [Create a recurring scheduled action that specifies a time zone](#recurring-schedule-set-time-zone)
  + [Describe scheduled actions](#describe-scheduled-actions)
  + [Delete a scheduled action](#delete-sch-actions-aws-cli)
+ [Limitations](#scheduled-scaling-limitations)

## Considerations<a name="sch-actions_rules"></a>

When you create a scheduled action, keep the following in mind:
+ A scheduled action sets the desired, minimum, and maximum sizes to what is specified by the scheduled action at the date and time specified\. The request can optionally include only one of these sizes\. For example, you can create a scheduled action with only the desired capacity specified\. In some cases, however, you must include the minimum and maximum sizes to ensure that the new desired capacity is not outside of these limits\.
+ By default, the times that you set are in Coordinated Universal Time \(UTC\)\. When specifying a recurring schedule with a cron expression using the AWS CLI or an SDK, you can change the time zone to correspond to your local time zone or a time zone for another part of your network\. When you specify a time zone that observes Daylight Saving Time \(DST\), the recurring action automatically adjusts for Daylight Saving Time\. 
+ If you specify a recurring schedule, you can specify a date and time for the start time, the end time, or both\. 
  + Choose your start and end times carefully\. The start time and end time must be set in UTC\. 
  + If you specify a start time, Amazon EC2 Auto Scaling performs the action at this time, and then performs the action based on the specified recurrence\.
  + If you specify an end time, the action stops repeating after this time\. A scheduled action does not persist in your account once it has reached its end time\.
+ If needed, you can disable and re\-enable scheduled scaling at any time\. For more information, see [Suspending and resuming scaling processes](as-suspend-resume-processes.md)\.
+ The order of execution for scheduled actions is guaranteed within the same group, but not for scheduled actions across groups\.
+ A scheduled action generally executes within seconds\. However, the action might be delayed for up to two minutes from the scheduled start time\. Because actions within an Auto Scaling group are executed in the order that they are specified, scheduled actions with scheduled start times close to each other can take longer to execute\.

**Warning**  
Currently, the time zone feature is available only if you use the AWS CLI or an SDK, and is not available from the console\. If you specify a recurring schedule with a time zone, the cron expression uses the specified time zone, which you can check by describing the scheduled action from the CLI or an SDK\. If you use the console to update a scheduled action with a recurring schedule, this update action deletes the time zone \(if one was set\), which defaults the cron expression to UTC\.

## Create and manage scheduled actions \(console\)<a name="create-sch-actions"></a>

You can create scheduled actions that scale one time only or that scale on a recurring schedule using the console\. Complete the following procedure to create a scheduled action to scale your Auto Scaling group\.

**To create a scheduled action for an Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Automatic scaling** tab, in **Scheduled actions**, choose **Create scheduled action**\.

1. Enter a **Name** for the scheduled action\.

1. For **Desired capacity**, **Min**, **Max**, choose the new desired size of the group and the new minimum and maximum capacity\.

1. For **Recurrence**, choose one of the available options\.
   + If you want to scale only once, choose **Once**\.
   + If you want to scale on a recurring basis, choose how often Amazon EC2 Auto Scaling should run the scheduled action\. 
     + If you choose an option that begins with **Every**, the cron expression is created for you\.
     + If you choose **Cron**, enter a cron expression that specifies when to perform the action, in UTC\. The cron expression format consists of five fields separated by white space\. For more information about this format, see [Crontab](http://crontab.org)\. For examples of cron expressions, see [https://crontab\.guru/examples\.html](https://crontab.guru/examples.html)\.

1. Define a **Start time**, in UTC\.
   + If you chose **Once** as the recurrence, the start time defines the date and time for the schedule action to run\. 
   + If you chose an option that runs on a recurring basis, the start time defines when the first scheduled action in the recurring series runs\. 

1.  \(Optional\) For recurrent actions, you can specify a value for **End time**, in UTC\.

1. Choose **Create**\.

   The console displays the scheduled actions for the Auto Scaling group\. 

**To create a scheduled action for an Auto Scaling group \(old console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Scheduled Actions** tab, choose **Create Scheduled Action**\.

1. On the **Create Scheduled Action** page, do the following:
   + Specify the size of the group using at least one of the following values: **Min**, **Max**, or **Desired Capacity**\.
   + For **Recurrence**, choose one of the available options\.
     + If you want to scale only once, choose **Once**\.
     + If you want to scale on a recurring basis, choose how often Amazon EC2 Auto Scaling should run the scheduled action\. 
       + If you choose an option that begins with **Every**, the cron expression is created for you\.
       + If you choose **Cron**, enter a cron expression that specifies when to perform the action, in UTC\. The cron expression format consists of five fields separated by white space\. For more information about this format, see [Crontab](http://crontab.org)\. For examples of cron expressions, see [https://crontab\.guru/examples\.html](https://crontab.guru/examples.html)\.
   + Define a **Start time**\. 
     + If you chose **Once** as the recurrence, the start time defines the date and time for the schedule action to run\. 
     + If you chose an option that runs on a recurring basis, the start time defines when the first scheduled action in the recurring series runs\. 
   + \(Optional\) If you specified a recurring schedule, you can specify a value for **End Time**\.

1. Choose **Create**\. 

   The console displays the scheduled actions for the Auto Scaling group\.

### Update a scheduled action<a name="update-sch-action"></a>

After creating a scheduled action, you can update any of its settings except the name and the time zone\. When you edit a scheduled action, all times are in UTC\.

**To update a scheduled action \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Automatic scaling** tab, in **Scheduled actions**, select a scheduled action\.

1. Choose **Actions**, **Edit**\.

1. Make the needed changes and choose **Save changes**\.

### Delete a scheduled action<a name="delete-sch-action"></a>

When you no longer need a scheduled action, you can delete it\.

**To delete a scheduled action \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select your Auto Scaling group\.

1. On the **Automatic scaling** tab, in **Scheduled actions**, select a scheduled action\.

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

## Create and manage scheduled actions \(AWS CLI\)<a name="create-sch-actions-aws-cli"></a>

You can create and update scheduled actions that scale one time only or that scale on a recurring schedule using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command\. 

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

To schedule scaling on a recurring schedule, use the `--recurrence "cron expression"` option\. The supported cron expression format consists of five fields separated by white spaces: \[Minute\] \[Hour\] \[Day\_of\_Month\] \[Month\_of\_Year\] \[Day\_of\_Week\]\. For more information about this format, see [Crontab](http://crontab.org)\. For examples of cron expressions, see [https://crontab\.guru/examples\.html](https://crontab.guru/examples.html)\.

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

To describe the scheduled actions for an Auto Scaling group, use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scheduled-actions.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scheduled-actions.html) command\.

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

To delete a scheduled action, use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-scheduled-action.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-scheduled-action.html) command\.

```
aws autoscaling delete-scheduled-action --auto-scaling-group-name my-asg \
  --scheduled-action-name my-recurring-action
```

## Limitations<a name="scheduled-scaling-limitations"></a>
+ The names of scheduled actions must be unique per Auto Scaling group\. 
+ A scheduled action must have a unique time value\. If you attempt to schedule an activity at a time when another scaling activity is already scheduled, the call is rejected and returns an error indicating that a scheduled action with this scheduled start time already exists\.
+ You can create a maximum of 125 scheduled actions per Auto Scaling group\.
+ You can only specify a time zone for a cron expression\.