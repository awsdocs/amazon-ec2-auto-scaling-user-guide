# Check the status of an instance refresh<a name="check-status-instance-refresh"></a>

After an instance refresh has begun, you can get its status using the AWS Management Console or AWS CLI\.

**Tip**  
In the following procedure, you look at the **Instance refresh history**, **Activity history**, and **Instances** sections for the Auto Scaling group\. In each, the named columns should already be displayed\. To display hidden columns or change the number of rows shown, choose the gear icon on the top right corner of each section to open the preferences modal\. Update the settings as needed and choose **Confirm**\.

**To check the status of an instance refresh \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom of the **Auto Scaling groups** page\.

1. On the **Instance refresh** tab, under **Instance refresh history**, you can determine the status of your request by looking at the **Status** column\. The operation goes into `Pending` status while it is initializing\. The status should then quickly change to `InProgress`\. When all instances are updated, the status changes to `Successful`\.

1. On the **Activity** tab, under **Activity history**, when the instance refresh starts, you see entries when instances are terminated and another set of entries when instances are launched\. In the **Description** column, you can find the instance ID\. 

1. \(Optional\) If you have numerous scaling activities, you can see more of them by choosing the **>** icon at the top of the activity history\.

1. On the **Instance management** tab, under **Instances**, you can verify that your instances launched successfully\. Initially, your instances are in the `Pending` state\. After an instance is ready to receive traffic, its state is `InService`\. The **Health status** column shows the result of the health checks on your instances\.

**To check the status of an instance refresh \(AWS CLI\)**  
View the instance refreshes for an Auto Scaling group by using the following [describe\-instance\-refreshes](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-instance-refreshes.html) command\.

```
aws autoscaling describe-instance-refreshes --auto-scaling-group-name my-asg
```

Example output:

```
{
    "InstanceRefreshes": [
        {
            "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b",
            "AutoScalingGroupName": "my-asg",
            "Status": "InProgress",
            "StatusReason": "Waiting for instances to warm up before continuing. For example: 0e69cc3f05f825f4f is warming up.",
            "EndTime": "2023-03-23T16:42:55Z",
            "PercentageComplete": 0,
            "InstancesToUpdate": 0,
            "Preferences": {
                "MinHealthyPercentage": 100,
                "InstanceWarmup": 300,
                "CheckpointPercentages": [
                    50
                ],
                "CheckpointDelay": 3600,
                "SkipMatching": false,
                "AutoRollback": true,
                "ScaleInProtectedInstances": "Ignore",
                "StandbyInstances": "Ignore"
            }
        },
        {
            "InstanceRefreshId": "dd7728d0-5bc4-4575-96a3-1b2c52bf8bb1",
            "AutoScalingGroupName": "my-asg",
            "Status": "Successful",
            "EndTime": "2022-06-02T16:53:37Z",
            "PercentageComplete": 100,
            "InstancesToUpdate": 0,
            "Preferences": {
                "MinHealthyPercentage": 90,
                "InstanceWarmup": 300,
                "SkipMatching": true,
                "AutoRollback": true,
                "ScaleInProtectedInstances": "Ignore",
                "StandbyInstances": "Ignore"
            }
        }
    ]
}
```

## Instance refresh statuses<a name="instance-refresh-statuses"></a>

When you start an instance refresh, it enters the **Pending** status\. It passes from **Pending** to **InProgress** until it reaches **Successful**, **Failed**, **Cancelled**, **RollbackSuccessful**, or **RollbackFailed**\.

An instance refresh can have the following statuses:


| Status | Description | 
| --- | --- | 
| Pending | The request was created, but the instance refresh has not started\. | 
| InProgress | An instance refresh is in progress\. | 
| Successful | An instance refresh completed successfully\. | 
| Failed | An instance refresh failed to complete\. You can troubleshoot using the status reason and the scaling activities\. | 
| Cancelling | An ongoing instance refresh is being cancelled\. | 
| Cancelled | The instance refresh is cancelled\. | 
| RollbackInProgress | An instance refresh is being rolled back\. | 
| RollbackFailed | The rollback failed to complete\. You can troubleshoot using the status reason and the scaling activities\. | 
| RollbackSuccessful | The rollback completed successfully\. | 