# Check the status of an instance refresh<a name="check-status-instance-refresh"></a>

You can get the status of an instance refresh for your Auto Scaling group using the console or the AWS CLI\.

**Tip**  
In the following procedure, you look at the **Instance refresh history**, **Activity history**, and **Instances** sections for the Auto Scaling group\. In each, the named columns should already be displayed\. To display hidden columns or change the number of rows shown, choose the gear icon on the top right corner of each section to open the preferences modal, update the settings as needed, and choose Confirm\.

**To check the status of an instance refresh \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom of the **Auto Scaling groups** page\.

1. On the **Instance refresh** tab, under **Instance refresh history**, you can determine the status of your request by looking at the **Status** column\. The operation goes into `Pending` status while it is initializing\. The status should then quickly change to `InProgress`\. When all instances are updated, the status changes to `Successful`\.

1. On the **Activity** tab, under **Activity history**, when the instance refresh starts, you see entries when instances are terminated and another set of entries when instances are launched\. In the **Description** column, you can find the instance ID\. 

1. \(Optional\) If you have a lot of scaling activities, you can choose the **>** icon at the top edge of the activity history to see the next page of scaling activities\.

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