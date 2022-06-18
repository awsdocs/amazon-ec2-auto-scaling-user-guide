# Troubleshoot Amazon EC2 Auto Scaling<a name="CHAP_Troubleshooting"></a>

Amazon EC2 Auto Scaling provides specific and descriptive errors to help you troubleshoot issues\. You can find the error messages in the description of the scaling activities\.

**Topics**
+ [Retrieve an error message from scaling activities](#RetrievingErrors)
+ [Additional troubleshooting resources](#additional-troubleshooting-resources)
+ [Troubleshoot Amazon EC2 Auto Scaling: EC2 instance launch failures](ts-as-instancelaunchfailure.md)
+ [Troubleshoot Amazon EC2 Auto Scaling: AMI issues](ts-as-ami.md)
+ [Troubleshoot Amazon EC2 Auto Scaling: Load balancer issues](ts-as-loadbalancer.md)
+ [Troubleshoot Amazon EC2 Auto Scaling: Launch templates](ts-as-launch-template.md)
+ [Troubleshoot Amazon EC2 Auto Scaling: Health checks](ts-as-healthchecks.md)

## Retrieve an error message from scaling activities<a name="RetrievingErrors"></a>

To retrieve an error message from the description of scaling activities, use the [describe\-scaling\-activities](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command\. You have a record of scaling activities that dates back 6 weeks\. Scaling activities are ordered by start time, with the latest scaling activities listed first\. 

**Note**  
The scaling activities are also displayed in the activity history in the Amazon EC2 Auto Scaling console on the **Activity** tab for the Auto Scaling group\.

To see the scaling activities for a specific Auto Scaling group, use the following command\. 

```
aws autoscaling describe-scaling-activities --auto-scaling-group-name my-asg
```

The following is an example response, where `StatusCode` contains the current status of the activity and `StatusMessage` contains the error message\.

```
{
    "Activities": [
        {
            "ActivityId": "3b05dbf6-037c-b92f-133f-38275269dc0f",
            "AutoScalingGroupName": "my-asg",
            "Description": "Launching a new EC2 instance: i-003a5b3ffe1e9358e.  Status Reason: Instance failed to complete user's Lifecycle Action: Lifecycle Action with token e85eb647-4fe0-4909-b341-a6c42d8aba1f was abandoned: Lifecycle Action Completed with ABANDON Result",
            "Cause": "At 2021-01-11T00:35:52Z a user request created an AutoScalingGroup changing the desired capacity from 0 to 1.  At 2021-01-11T00:35:53Z an instance was started in response to a difference between desired and actual capacity, increasing the capacity from 0 to 1.",
            "StartTime": "2021-01-11T00:35:55.542Z",
            "EndTime": "2021-01-11T01:06:31Z",
            "StatusCode": "Cancelled",
            "StatusMessage": "Instance failed to complete user's Lifecycle Action: Lifecycle Action with token e85eb647-4fe0-4909-b341-a6c42d8aba1f was abandoned: Lifecycle Action Completed with ABANDON Result",
            "Progress": 100,
            "Details": "{\"Subnet ID\":\"subnet-5ea0c127\",\"Availability Zone\":\"us-west-2b\"...}",
            "AutoScalingGroupARN": "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:283179a2-f3ce-423d-93f6-66bb518232f7:autoScalingGroupName/my-asg"
        },
     ...
    ]
}
```

For a description of the fields in the output, see [Activity](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_Activity.html) in the *Amazon EC2 Auto Scaling API Reference*\.

To view scaling activities for a deleted Auto Scaling group, add the `--include-deleted-groups` option to the [describe\-scaling\-activities](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command as follows\. 

```
aws autoscaling describe-scaling-activities --auto-scaling-group-name my-asg --include-deleted-groups
```

The following is an example response, with a scaling activity for a deleted group\. 

```
{
    "Activities": [
        {
            "ActivityId": "e1f5de0e-f93e-1417-34ac-092a76fba220",
            "AutoScalingGroupName": "my-asg",
            "Description": "Launching a new EC2 instance.  Status Reason: Your Spot request price of 0.001 is lower than the minimum required Spot request fulfillment price of 0.0031. Launching EC2 instance failed.",
            "Cause": "At 2021-01-13T20:47:24Z a user request update of AutoScalingGroup constraints to min: 1, max: 5, desired: 3 changing the desired capacity from 0 to 3.  At 2021-01-13T20:47:27Z an instance was started in response to a difference between desired and actual capacity, increasing the capacity from 0 to 3.",
            "StartTime": "2021-01-13T20:47:30.094Z",
            "EndTime": "2021-01-13T20:47:30Z",
            "StatusCode": "Failed",
            "StatusMessage": "Your Spot request price of 0.001 is lower than the minimum required Spot request fulfillment price of 0.0031. Launching EC2 instance failed.",
            "Progress": 100,
            "Details": "{\"Subnet ID\":\"subnet-5ea0c127\",\"Availability Zone\":\"us-west-2b\"...}",
            "AutoScalingGroupState": "Deleted",
            "AutoScalingGroupARN": "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:283179a2-f3ce-423d-93f6-66bb518232f7:autoScalingGroupName/my-asg"
        },
     ...
    ]
}
```

## Additional troubleshooting resources<a name="additional-troubleshooting-resources"></a>

The following pages provide additional information for troubleshooting issues with Amazon EC2 Auto Scaling\.
+ [Quotas for Amazon EC2 Auto Scaling](ec2-auto-scaling-quotas.md) 
+ [Verify a scaling activity for an Auto Scaling group](as-verify-scaling-activity.md) 
+ [View monitoring graphs in the Amazon EC2 Auto Scaling console](viewing-monitoring-graphs.md) 
+ [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md) 
+ [Considerations and limitations for lifecycle hooks](lifecycle-hooks.md#lifecycle-hook-considerations)
+  [Complete a lifecycle action](completing-lifecycle-hooks.md)
+  [Provide network connectivity for your Auto Scaling instances using Amazon VPC](asg-in-vpc.md) 
+ [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md) 
+ [Disable a scaling policy for an Auto Scaling group](as-enable-disable-scaling-policy.md) 
+  [Suspend and resume a process for an Auto Scaling group](as-suspend-resume-processes.md) 
+ [Control which Auto Scaling instances terminate during scale in](as-instance-termination.md) 
+ [Delete your Auto Scaling infrastructure](as-process-shutdown.md) 

The following AWS resources can also be of help:
+ [Amazon EC2 Auto Scaling topics in the AWS Knowledge Center](https://aws.amazon.com/premiumsupport/knowledge-center/#AWS_Auto_Scaling) 
+ [Amazon EC2 Auto Scaling questions on AWS re:Post](https://repost.aws/tags/TA5Ef3s6KtTiqT0mCRhR79ig/amazon-ec-2-auto-scaling)
+ [Amazon EC2 Auto Scaling posts in the AWS Compute Blog](http://aws.amazon.com/blogs/compute/category/compute/auto-scaling/) 
+ [Troubleshooting CloudFormation in the *AWS CloudFormation User Guide*](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html) 

Troubleshooting often requires iterative query and discovery by an expert or from a community of helpers\. If you continue to experience issues after trying the suggestions in this section, contact AWS Support \(in the AWS Management Console, click **Support**, **Support Center**\) or ask a question on [AWS re:Post](https://repost.aws/) using the **Amazon EC2 Auto Scaling** tag\.