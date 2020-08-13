# Scheduled scaling for Amazon EC2 Auto Scaling<a name="schedule_time"></a>

Scheduled scaling allows you to set your own scaling schedule\. For example, let's say that every week the traffic to your web application starts to increase on Wednesday, remains high on Thursday, and starts to decrease on Friday\. You can plan your scaling actions based on the predictable traffic patterns of your web application\. Scaling actions are performed automatically as a function of time and date\. 

**Note**  
For scaling based on predictable load changes, you can also use the predictive scaling feature of AWS Auto Scaling\. For more information, see the [AWS Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/plans/userguide/)\.

To configure your Auto Scaling group to scale based on a schedule, you create a scheduled action\. The scheduled action tells Amazon EC2 Auto Scaling to perform a scaling action at specified times\. To create a scheduled scaling action, you specify the start time when the scaling action should take effect, and the new minimum, maximum, and desired sizes for the scaling action\. At the specified time, Amazon EC2 Auto Scaling updates the group with the values for minimum, maximum, and desired size that are specified by the scaling action\.

You can create scheduled actions for scaling one time only, or for scaling on a recurring schedule\.

## Considerations<a name="sch-actions_rules"></a>

When you create a scheduled action, keep the following in mind:
+ A scheduled action sets the minimum, maximum, and desired sizes to what is specified by the scheduled action at the time specified by the scheduled action\. It does not keep track of old values and return to the older values after the end time\.
+ A scheduled action generally executes within seconds\. However, the action might be delayed for up to two minutes from the scheduled start time\. Because actions within an Auto Scaling group are executed in the order that they are specified, scheduled actions with scheduled start times close to each other can take longer to execute\.
+ The order of execution for scheduled actions is guaranteed within the same group, but not for scheduled actions across groups\.
+ A scheduled action must have a unique time value\. If you attempt to schedule an activity at a time when another scaling activity is already scheduled, the call is rejected with an error message noting the conflict\.
+ You can create a maximum of 125 scheduled actions per Auto Scaling group\.
+ A scheduled action does not persist in your account once it has reached its end time\.
+ You can temporarily disable scheduled scaling without deleting your scheduled actions\. For more information, see [Suspending and resuming scaling processes](as-suspend-resume-processes.md)\.
+ Cooldown periods are not supported\. 
+ You can also schedule scaling actions for resources beyond Amazon EC2\. For more information, see [Scheduled scaling](https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling-scheduled-scaling.html) in the *Application Auto Scaling User Guide*\.

## Create and manage scheduled actions \(console\)<a name="create-sch-actions"></a>

You can create scheduled actions that scale one time only or that scale on a recurring schedule using the console\. Complete the following procedure to create a scheduled action to scale your Auto Scaling group\.

**To create a scheduled action for an Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Automatic scaling** tab, in **Scheduled actions**, choose **Create scheduled action**\. \(Old console: The **Scheduled Actions** tab is where you can create a scheduled action\.\) 

1. In the **Create scheduled action** dialog box, do the following:
   + For **Name**, enter a name for the scheduled action\.
   + Specify the size of the group using at least one of the following values: **Min**, **Max**, or **Desired capacity**\.
   + Choose an option for **Recurrence**\. If you choose **Once**, the action is performed at the specified time\. If you choose **Cron**, enter a cron expression that specifies when to perform the action, in UTC\. If you choose an option that begins with **Every**, the cron expression is created for you\.
   + If you chose **Once** for **Recurrence**, specify the date and time for the action to run in **Start time**\.
   + For recurrent actions, you can specify values for both **Start time** and **End time**\. If you specify a start time, the earliest time that the action will be performed is at this time\. If you specify an end time, the action stops repeating after this time\. 

1. Choose **Create**\.

### Update a scheduled action<a name="update-sch-action"></a>

If your requirements change, you can update a scheduled action\.

**To update a scheduled action**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Automatic scaling** tab, in **Scheduled actions**, select a scheduled action\. \(Old console: The **Scheduled Actions** tab is where you can select the scheduled action\.\) 

1. Choose **Actions**, **Edit**\.

1. In the **Edit scheduled action** dialog box, do the following:
   + Update the size of the group as needed using **Min**, **Max**, or **Desired capacity**\.
   + Update the specified recurrence as needed\.
   + Update the start and end time as needed\.

1. Choose **Save changes**\.

### Delete a scheduled action<a name="delete-sch-action"></a>

When you no longer need a scheduled action, you can delete it\.

**To delete a scheduled action**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Automatic scaling** tab, in **Scheduled actions**, select a scheduled action\. \(Old console: The **Scheduled Actions** tab is where you can select the scheduled action\.\) 

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

## Create and manage scheduled actions \(AWS CLI\)<a name="create-sch-actions-aws-cli"></a>

You can create and update scheduled actions that scale one time only or that scale on a recurring schedule using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command\. 

**To scale one time only**  
You can specify a one\-time schedule to automatically scale your Auto Scaling group at a certain date and time, in UTC\. 
+ To decrease the number of running instances in your Auto Scaling group at a specific time, use the following command\. At the date and time specified for `--start-time`, if the group currently has more than 1 instance, the group scales in to 1 instance\. 

  ```
  aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-one-time-action \
    --auto-scaling-group-name my-asg --start-time "2019-05-13T08:00:00Z" --desired-capacity 1
  ```
+ To increase the number of running instances in your Auto Scaling group at a specific time, use the following command\. At the date and time specified for `--start-time`, if the group currently has fewer than 3 instances, the group scales out to 3 instances\. 

  ```
  aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-one-time-action \
    --auto-scaling-group-name my-asg --start-time "2019-05-12T08:00:00Z" --desired-capacity 3
  ```

**To scale on a recurring schedule**  
You can specify a recurrence schedule, in UTC, using the Unix cron syntax format\. This format consists of five fields separated by white spaces: \[Minute\] \[Hour\] \[Day\_of\_Month\] \[Month\_of\_Year\] \[Day\_of\_Week\]\. For more information about this format, see [Crontab](http://crontab.org)\. 

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command to create a scheduled action that runs at 00:30 hours on the first of January, June, and December each year\.

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name my-recurring-action \
  --auto-scaling-group-name my-asg --recurrence "30 0 1 1,6,12 *" --desired-capacity 3
```

### Delete a scheduled action<a name="delete-sch-actions-aws-cli"></a>

**To delete a scheduled action**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-scheduled-action.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-scheduled-action.html) command\.

```
aws autoscaling delete-scheduled-action --scheduled-action-name my-recurring-action
```