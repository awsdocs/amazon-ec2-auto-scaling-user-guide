# Scheduled Scaling<a name="schedule_time"></a>

Scaling based on a schedule allows you to scale your application in response to predictable load changes\. For example, every week the traffic to your web application starts to increase on Wednesday, remains high on Thursday, and starts to decrease on Friday\. You can plan your scaling activities based on the predictable traffic patterns of your web application\. 

To configure your Auto Scaling group to scale based on a schedule, you create a scheduled action, which tells Amazon EC2 Auto Scaling to perform a scaling action at specified times\. To create a scheduled scaling action, you specify the start time when you want the scaling action to take effect, and the new minimum, maximum, and desired sizes for the scaling action\. At the specified time, Amazon EC2 Auto Scaling updates the group with the values for minimum, maximum, and desired size specified by the scaling action\.

You can create scheduled actions for scaling one time only or for scaling on a recurring schedule\.


+ [Considerations for Scheduled Actions](#sch-actions_rules)
+ [Create a Scheduled Action Using the Console](#create-sch-actions)
+ [Update a Scheduled Action](#update-sch-action)
+ [Create or Update a Scheduled Action Using the AWS CLI](#create-sch-actions-aws-cli)
+ [Delete a Scheduled Action](#delete-sch-action)

## Considerations for Scheduled Actions<a name="sch-actions_rules"></a>

When you create a scheduled action, keep the following in mind\.

+ The order of execution for scheduled actions is guaranteed within the same group, but not for scheduled actions across groups\.

+ A scheduled action generally executes within seconds\. However, the action may be delayed for up to two minutes from the scheduled start time\. Because actions within an Auto Scaling group are executed in the order they are specified, scheduled actions with scheduled start times close to each other can take longer to execute\.

+ You can create a maximum of 125 scheduled actions per Auto Scaling group\.

+ A scheduled action must have a unique time value\. If you attempt to schedule an activity at a time when another scaling activity is already scheduled, the call is rejected with an error message noting the conflict\.

+ Cooldown periods are not supported\.

## Create a Scheduled Action Using the Console<a name="create-sch-actions"></a>

Complete the following procedure to create a scheduled action to scale your Auto Scaling group\.

**To create a scheduled action**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Scheduled Actions** tab, choose **Create Scheduled Action**\.

1. On the **Create Scheduled Action** page, do the following:

   + Specify the size of the group using at least one of **Min**, **Max**, and **Desired Capacity**\.

   + Choose an option for **Recurrence**\. If you choose **Once**, the action is performed at the specified time\. If you select **Cron**, type a Cron expression that specifies when to perform the action, in UTC\. If you select an option that begins with **Every**, the Cron expression is created for you\.

   + If you chose **Once** for **Recurrence**, specify the time for the action in **Start Time**\.

   + If you specified a recurring schedule, you can specify values for **Start Time** and **End Time**\. If you specify a start time, the action is performed at this time, and then performs the action based on the recurring schedule\. If you specify an end time, the action is not performed after this time\.

1. Choose **Create**\.

## Update a Scheduled Action<a name="update-sch-action"></a>

If your requirements change, you can update a scheduled action\.

**To update a scheduled action**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Scheduled Actions** tab, select the scheduled action\.

1. Choose **Actions**, **Edit**\.

1. On the **Edit Scheduled Action** page, do the following:

   + Update the size of the group as needed using **Min**, **Max**, or **Desired Capacity**\.

   + Update the specified recurrence as needed\.

   + Update the start and end time as needed\.

   + Choose **Save**\.

## Create or Update a Scheduled Action Using the AWS CLI<a name="create-sch-actions-aws-cli"></a>

You can create a schedule for scaling one time only or for scaling on a recurring schedule\.

**To schedule scaling for one time only**  
To increase the number of running instances in your Auto Scaling group at a specific time, in "YYYY\-MM\-DDThh:mm:ssZ" format in UTC, use the following [put\-scheduled\-update\-group\-action](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command:

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name ScaleUp --auto-scaling-group-name my-asg --start-time "2013-05-12T08:00:00Z" --desired-capacity 3 
```

To decrease the number of running instances in your Auto Scaling group at a specific time, in "YYYY\-MM\-DDThh:mm:ssZ" format in UTC, use the following `put-scheduled-update-group-action` command:

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name ScaleDown --auto-scaling-group-name my-asg --start-time "2013-05-13T08:00:00Z" --desired-capacity 1 
```

**To schedule scaling on a recurring schedule**  
You can specify a recurrence schedule, in UTC, using the Cron format\. For more information, see the [Cron](http://en.wikipedia.org/wiki/Cron) Wikipedia entry\.

Use the following [put\-scheduled\-update\-group\-action](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scheduled-update-group-action.html) command to create a scheduled action that runs at 00:30 hours on the first of January, June, and December each year:

```
aws autoscaling put-scheduled-update-group-action --scheduled-action-name scaleup-schedule-year --auto-scaling-group-name my-asg --recurrence "30 0 1 1,6,12 0" --desired-capacity 3 
```

## Delete a Scheduled Action<a name="delete-sch-action"></a>

When you are finished with a scheduled action, you can delete it\.

**To delete a scheduled action using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Scheduled Actions** tab, select the scheduled action\.

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete a scheduled action using the AWS CLI**  
Use the following [delete\-scheduled\-action](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-scheduled-action.html) command:

```
aws autoscaling delete-scheduled-action --scheduled-action-name ScaleUp
```