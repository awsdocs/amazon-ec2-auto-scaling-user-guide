# Troubleshooting Amazon EC2 Auto Scaling<a name="CHAP_Troubleshooting"></a>

Amazon EC2 Auto Scaling provides specific and descriptive errors to help you troubleshoot issues\. You can find the error messages in the description of the scaling activities\.

**Topics**
+ [Retrieving an error message from scaling activities](#RetrievingErrors)
+ [Troubleshooting Amazon EC2 Auto Scaling: EC2 instance launch failures](ts-as-instancelaunchfailure.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: AMI issues](ts-as-ami.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: Load balancer issues](ts-as-loadbalancer.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: Launch templates](ts-as-launch-template.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: Health checks](ts-as-healthchecks.md)

## Retrieving an error message from scaling activities<a name="RetrievingErrors"></a>

To retrieve an error message from the description of scaling activities, use the [describe\-scaling\-activities](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command\. You have a record of scaling activities that dates back 6 weeks\. Scaling activities are ordered by start time, with the latest scaling activities listed first\. 

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

The following tables list the types of error messages and provide links to the troubleshooting resources that you can use to troubleshoot issues\.


**Launch issues**  

| Issue | Error message | 
| --- | --- | 
| Availability Zone |  [The requested Availability Zone is no longer supported\. Please retry your request\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-5)  | 
| Block device mapping |  [Invalid device name <device name> / Invalid device name upload\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-8)  | 
| Block device mapping |  [Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-9)  | 
| Block device mapping |  [EBS block device mappings not supported for instance\-store AMIs\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-10)  | 
| Instance configuration |  [The requested configuration is currently not supported\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-3)  | 
| Instance type and Availability Zone |  [Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-6)  | 
| Insufficient instance capacity in Availability Zone |  [We currently do not have sufficient <instance type> capacity in the Availability Zone you requested\.\.\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-capacity-1)  | 
| Insufficient instance capacity for a Spot request |  [There is no Spot capacity available that matches your request\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-capacity-2)  | 
| Key pair |  [The key pair <key pair associated with your EC2 instance> does not exist\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-2)  | 
| Placement group |  [Placement groups may not be used with instances of type 'm1\.large'\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-11)  | 
| Quota limits |  [<number of instances> instance\(s\) are already running\. Launching EC2 instance failed\. ](ts-as-instancelaunchfailure.md#ts-as-capacity-3)  | 
| Security group |  [The security group <name of the security group> does not exist\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-1)  | 
| Service\-linked role |  [Client\.InternalError: Client error on launch\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-12)  | 
| Spot price too low |  [Your Spot request price of 0\.015 is lower than the minimum required Spot request fulfillment price of 0\.0735\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-7)  | 


**AMI issues**  

| Issue | Error message | 
| --- | --- | 
| AMI ID |  [The AMI ID <ID of your AMI> does not exist\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-1)  | 
| AMI ID |  [AMI <AMI ID> is pending, and cannot be run\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-2)  | 
| AMI ID |  [Value \(<ami ID>\) for parameter virtualName is invalid\.](ts-as-ami.md#ts-as-ami-4)  | 
| Architecture mismatch |  [The requested instance type's architecture \(i386\) does not match the architecture in the manifest for ami\-6622f00f \(x86\_64\)\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-5)  | 


**Load balancer issues**  

| Issue | Error message | 
| --- | --- | 
| Cannot find load balancer |  [Cannot find Load Balancer <your launch environment>\. Validating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-1)  | 
| Instances in VPC |  [EC2 instance <instance ID> is not in VPC\. Updating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-3)  | 
| No active load balancer |  [There is no ACTIVE Load Balancer named <load balancer name>\. Updating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-2)  | 