# View health check status and the reason for health check failures<a name="warm-pools-health-checks-monitor-view-status"></a>

Health checks allow Amazon EC2 Auto Scaling to determine when an instance is unhealthy and should be terminated\. For warm pool instances kept in a `Stopped` state, it employs the knowledge that Amazon EBS has of a `Stopped` instance's availability to identify unhealthy instances\. It does this by calling the `DescribeVolumeStatus` API to determine the status of the EBS volume that's attached to the instance\. For warm pool instances kept in a `Running` state, it relies on EC2 status checks to determine instance health\. While there is no health check grace period for warm pool instances, Amazon EC2 Auto Scaling doesn't start checking instance health until the lifecycle hook finishes\. 

When an instance is found to be unhealthy, Amazon EC2 Auto Scaling automatically deletes the unhealthy instance and creates a new one to replace it\. Instances are usually terminated within a few minutes after failing their health check\. For more information, see [Replace unhealthy instances](ec2-auto-scaling-health-checks.md#replace-unhealthy-instance)\.

Custom health checks are also supported\. This can be helpful if you have your own health check system that can detect an instance's health and send this information to Amazon EC2 Auto Scaling\. For more information, see [Custom health detection tasks](ec2-auto-scaling-health-checks.md#as-configure-healthcheck)\.

On the Amazon EC2 Auto Scaling console, you can view the status \(healthy or unhealthy\) of your warm pool instances\. You can also view their health status using the AWS CLI or one of the SDKs\. 

**To view the status of your warm pool instances \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Instance management** tab, under **Warm pool instances**, the **Lifecycle** column contains the state of your instances\.

   The **Health status** column shows the assessment that Amazon EC2 Auto Scaling has made of instance health\.
**Note**  
New instances start healthy\. Until the lifecycle hook is finished, an instance's health is not checked\.

**To view the reason for health check failures \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched or terminated instances\.

   If it terminated any unhealthy instances, the **Cause** column shows the date and time of the termination and the reason for the health check failure\. For example, "At 2021\-04\-01T21:48:35Z an instance was taken out of service in response to EBS volume health check failure"\. 

**To view the status of your warm pool instances \(AWS CLI\)**  
View the warm pool for an Auto Scaling group by using the following [describe\-warm\-pool](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-warm-pool.html) command\.

```
aws autoscaling describe-warm-pool --auto-scaling-group-name my-asg
```

Example output\.

```
{
    "WarmPoolConfiguration": {
        "MinSize": 0,
        "PoolState": "Stopped"
    },
    "Instances": [
        {
            "InstanceId": "i-0b5e5e7521cfaa46c",
            "InstanceType": "t2.micro",
            "AvailabilityZone": "us-west-2a",
            "LifecycleState": "Warmed:Stopped",
            "HealthStatus": "Healthy",
            "LaunchTemplate": {
                "LaunchTemplateId": "lt-08c4cd42f320d5dcd",
                "LaunchTemplateName": "my-template-for-auto-scaling",
                "Version": "1"
            }
        },
        {
            "InstanceId": "i-0e21af9dcfb7aa6bf",
            "InstanceType": "t2.micro",
            "AvailabilityZone": "us-west-2a",
            "LifecycleState": "Warmed:Stopped",
            "HealthStatus": "Healthy",
            "LaunchTemplate": {
                "LaunchTemplateId": "lt-08c4cd42f320d5dcd",
                "LaunchTemplateName": "my-template-for-auto-scaling",
                "Version": "1"
            }
        }
    ]
}
```

**To view the reason for health check failures \(AWS CLI\)**  
Use the following [describe\-scaling\-activities](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command\. 

```
aws autoscaling describe-scaling-activities --auto-scaling-group-name my-asg
```

The following is an example response, where `Description` indicates that your Auto Scaling group has terminated an instance and `Cause` indicates the reason for the health check failure\. 

Scaling activities are ordered by start time\. Activities still in progress are described first\. 

```
{
  "Activities": [
    {
      "ActivityId": "4c65e23d-a35a-4e7d-b6e4-2eaa8753dc12",
      "AutoScalingGroupName": "my-asg",
      "Description": "Terminating EC2 instance: i-04925c838b6438f14",
      "Cause": "At 2021-04-01T21:48:35Z an instance was taken out of service in response to EBS volume health check failure.",
      "StartTime": "2021-04-01T21:48:35.859Z",
      "EndTime": "2021-04-01T21:49:18Z",
      "StatusCode": "Successful",
      "Progress": 100,
      "Details": "{\"Subnet ID\":\"subnet-5ea0c127\",\"Availability Zone\":\"us-west-2a\"...}",
      "AutoScalingGroupARN": "arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:283179a2-f3ce-423d-93f6-66bb518232f7:autoScalingGroupName/my-asg"
    },
...
  ]
}
```