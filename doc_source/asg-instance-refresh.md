# Replacing Auto Scaling instances based on an instance refresh<a name="asg-instance-refresh"></a>

When a configuration change requires replacing instances, and you have a large number of instances in your Auto Scaling group, it can be difficult to manually replace instances a few at a time\. With an instance refresh, it's easier to update the instances in your Auto Scaling group\. 

During an instance refresh, Amazon EC2 Auto Scaling takes a set of instances out of service, terminates them, and then launches a set of instances with the new configuration\. After that, instances are replaced on a rolling basis\. In a rolling update, when a new instance launches, Amazon EC2 Auto Scaling waits until the instance passes a health check and completes warm\-up, before moving on to replace another instance\. This process repeats until all instances are replaced\. 

This feature is helpful, for example, when you have a new launch template or launch configuration that specifies a new AMI or new user data\. You just need to update your Auto Scaling group to specify the new launch template or launch configuration\. Then start an instance refresh to immediately begin the process of replacing all instances in the group\.

Instance refreshes depend on health checks to determine whether your application is healthy enough to consider a replacement successful\. For more information, see [Health checks for Auto Scaling instances](healthcheck.md)\. 

Before starting an instance refresh, you can configure the minimum healthy percentage to control the level of disruption to your application\. If your application doesn't pass health checks, the rolling update process waits for a time period of up to 60 minutes after it reaches the minimum healthy threshold before it eventually fails\. The intention is to give it time to recover in case of a temporary issue\. If the rolling update process fails, any instances that were already replaced are not rolled back to their previous configuration\. To fix a failed instance refresh, first resolve the underlying issue that caused the update to fail, and then initiate another instance refresh\. 

After determining that a newly launched instance is healthy, Amazon EC2 Auto Scaling does not immediately move on to the next replacement\. It provides a window for each instance to warm up after launching, which you can configure\. This can be helpful when you have configuration scripts that take time to run\. To protect your application's availability, ensure that the instance warm\-up period covers the expected startup time for your application, from when a new instance comes into service to when it can receive traffic\. 

The following are things to consider when starting an instance refresh, to help ensure that the group continues to perform as expected\. 
+ While warming up, a newly launched instance is not counted toward the aggregated metrics of the Auto Scaling group\. 
+ If you added scaling policies to the Auto Scaling group, the scaling activities run in parallel\. If you set a long interval for the instance refresh warm\-up period, it will take more time for newly launched instances to be reflected in the metrics\. An adequate warm\-up period therefore helps to prevent Amazon EC2 Auto Scaling from scaling on stale metric data\. 
+ If you added a lifecycle hook to the Auto Scaling group, the warm\-up period does not start until the lifecycle hook actions complete and the instance enters the `InService` state\. 

You can start or cancel an instance refresh using the AWS Management Console, the AWS CLI, or an AWS SDK\. You can cancel an instance refresh anytime, but any instances that have already been replaced are not rolled back to their previous configuration\. 

## Start or cancel an instance refresh<a name="instance-refresh-configure"></a>

Before you begin, make sure that your Auto Scaling group is already associated with a new launch template or launch configuration\. 

**To start an instance refresh \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Instance refresh** tab, in **Instance refreshes**, choose **Start instance refresh**\.

1. In the **Start instance refresh** dialog box, do the following:
   + \(Optional\) For **Minimum healthy percentage**, keep the default value, `90`, or specify a new value within the range of 0 percent to 100 percent\. This is the amount of capacity in the Auto Scaling group that must remain healthy during an instance refresh to allow the operation to continue, expressed as a percentage of the group's desired capacity\. Increasing this setting to 100 percent limits the rate of replacement to one instance at a time\. In contrast, decreasing this setting to 0 percent has the effect of replacing all instances at the same time\. 
   + \(Optional\) For **Instance warmup**, the default is the value specified for the health check grace period for the group\. You can change the value to ensure that it reflects your actual application startup time\. The default value may not reflect very recent updates in the latest version of your application\. 

1. Choose **Start**\. 

**To check the status of an instance refresh \(new console\)**

1. On the **Instance refresh** tab, under **Instance refreshes**, you can determine the status of your request by looking at the **Status** column\. The operation goes into `Pending` status while it is initializing\. The status should then quickly change to `InProgress`\. When all instances are updated, the status changes to `Successful`\.

   During an instance refresh, if Amazon EC2 Auto Scaling is unable to replace any more instances, it provides a status message to help you resolve the issue\. If Amazon EC2 Auto Scaling determines that an instance is on standby or protected from scale in, it cannot replace the instance, but it will continue to replace other instances\. For any instances that it initially fails to replace, it keeps trying for a period of time\. Eventually, if the issue isn't resolved, it stops trying and the status changes to `Failed`\.

   The maximum amount of time that an instance refresh can remain actively replacing instances is 14 days\. 

1. On the **Activity** tab, under **Activity history**, when the instance refresh starts, you see entries when instances are terminated and another set of entries when instances are launched\. In the **Description** column, you can find the instance ID\. 

1. On the **Instance management** tab, under **Instances**, you can verify that your instances launched successfully\. Initially, your instances are in the `Pending` state\. After an instance is ready to receive traffic, its state is `InService`\. The **Health status** column shows the result of the health checks on your instances\.

**To cancel an instance refresh \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to the Auto Scaling group\.

1. On the **Instance refresh** tab, in **Instance refreshes**, choose **Cancel instance refresh**\.

1. When prompted for confirmation, choose **Confirm**\.

**To start an instance refresh \(AWS CLI\)**  
To start an instance refresh from the AWS CLI, use the [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command\. You can specify any preferences that you want to change in a JSON configuration file\. When you reference the configuration file, provide the file's path and name as shown in the following example\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`\.

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 400,
      "MinHealthyPercentage": 50
    }
}
```

Alternatively, you can start the instance refresh without the optional preferences by running the following command\. If preferences are not provided, the default values are used for `InstanceWarmup` and `MinHealthyPercentage`\.

```
aws autoscaling start-instance-refresh --auto-scaling-group-name my-asg
```

Example output\.

```
{
    "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
}
```

**To check the status of an instance refresh \(AWS CLI\)**  
View the instance refreshes for an Auto Scaling group by using the following [describe\-instance\-refreshes](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-instance-refreshes.html) command\.

```
aws autoscaling describe-instance-refreshes --auto-scaling-group-name my-asg
```

Example output\.

```
{
    "InstanceRefreshes": [
        {
            "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b",
            "AutoScalingGroupName": "my-asg",
            "Status": "InProgress",
            "StartTime": "2020-06-02T18:11:27Z",
            "PercentageComplete": 0,
            "InstancesToUpdate": 5
        },
        {
            "InstanceRefreshId": "dd7728d0-5bc4-4575-96a3-1b2c52bf8bb1",
            "AutoScalingGroupName": "my-asg",
            "Status": "Successful",
            "StartTime": "2020-06-02T16:43:19Z",
            "EndTime": "2020-06-02T16:53:37Z",
            "PercentageComplete": 100,
            "InstancesToUpdate": 0
        }
    ]
}
```

**To cancel an instance refresh \(AWS CLI\)**  
When you cancel an instance refresh using the [cancel\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/cancel-instance-refresh.html) command from the AWS CLI, specify the name of the Auto Scaling group as shown in the following example\. 

```
aws autoscaling cancel-instance-refresh --auto-scaling-group-name my-asg
```

Example output\.

```
{
    "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
}
```