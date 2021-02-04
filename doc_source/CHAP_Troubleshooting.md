# Troubleshooting Amazon EC2 Auto Scaling<a name="CHAP_Troubleshooting"></a>

Amazon EC2 Auto Scaling provides specific and descriptive errors to help you troubleshoot issues\. You can find the error messages in the description of the scaling activities\.

**Topics**
+ [General troubleshooting issues](#troubleshooting-general)
+ [Retrieving an error message](#RetrievingErrors)
+ [Troubleshooting Amazon EC2 Auto Scaling: EC2 Instance launch failures](ts-as-instancelaunchfailure.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: AMI issues](ts-as-ami.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: Load balancer issues](ts-as-loadbalancer.md)

## General troubleshooting issues<a name="troubleshooting-general"></a>

**Topics**
+ [Troubleshooting launch template error: "You are not authorized to perform this operation"](#ts-launch-template-unauthorized-error)
+ [Troubleshooting instance termination](#ts-failed-status-checks)

### Troubleshooting launch template error: "You are not authorized to perform this operation"<a name="ts-launch-template-unauthorized-error"></a>

The following sections can help you troubleshoot the "You are not authorized to perform this operation" error when creating or updating Auto Scaling groups\.

#### Permissions required for a launch template are missing<a name="launch-template-permissions"></a>

If you are attempting to use a launch template, and the IAM credentials you are using do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. For information about the permissions necessary to work with launch templates, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

#### Permissions required to pass an IAM role are missing<a name="ts-instance-profile-permissions"></a>

If you are attempting to use a launch template that specifies an instance profile, you must have permission to pass the IAM role that is associated with the instance profile\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\. For further troubleshooting topics related to instance profiles, see [Troubleshooting Amazon EC2 and IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_iam-ec2.html) in the *IAM User Guide*\.

### Troubleshooting instance termination<a name="ts-failed-status-checks"></a>

Amazon EC2 performs status checks on running EC2 instances to identify hardware and software issues\. If one or more checks fail, the instance returns a status of `impaired` to Amazon EC2 Auto Scaling, which then automatically replaces it as part of its health checks\. To determine why Amazon EC2 Auto Scaling terminated an instance, see [Why did Amazon EC2 Auto Scaling terminate an instance?](http://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-instance-how-terminated/) in the AWS Knowledge Center\.

Note that instances launched in an Auto Scaling group require sufficient warm\-up time \(grace period\) to prevent early termination due to a health check replacement\. To update the health check grace period for your Auto Scaling group to an appropriate time period for your application, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\. Make sure that the health check grace period is longer than the instance startup time\. Otherwise, healthy instances can be terminated before they finish starting up\. 

See also:
+ For more information, see [Health checks for Auto Scaling instances](healthcheck.md)\.
+ For a list of issues that prevent Amazon EC2 Auto Scaling from terminating an unhealthy instance, see [Why didn’t Amazon EC2 Auto Scaling terminate an unhealthy instance?](http://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-terminate-instance/) in the AWS Knowledge Center\.
+ For information about troubleshooting failed status checks, see [Troubleshooting instances with failed status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstances.html) in the *Amazon EC2 User Guide for Linux Instances*\. For help with Windows instances, see [Troubleshooting Windows Instances](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/troubleshooting-windows-instances.html) in the *Amazon EC2 User Guide for Windows Instances*\. 

## Retrieving an error message<a name="RetrievingErrors"></a>

To retrieve an error message from the description of scaling activities, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-scaling-activities.html) command as follows:

```
aws autoscaling describe-scaling-activities --auto-scaling-group-name my-asg
```

The following is an example response, where `StatusCode` contains the current status of the activity and `StatusMessage` contains the error message:

```
{
    "Activities": [
        {
            "Description": "Launching a new EC2 instance: i-4ba0837f",
            "AutoScalingGroupName": "my-asg",
            "ActivityId": "f9f2d65b-f1f2-43e7-b46d-d86756459699",
            "Details": "{"Availability Zone":"us-west-2c"}",
            "StartTime": "2013-08-19T20:53:29.930Z",
            "Progress": 100,
            "EndTime": "2013-08-19T20:54:02Z",
            "Cause": "At 2013-08-19T20:53:25Z a user request created an AutoScalingGroup...",
            "StatusCode": "Failed",
            "StatusMessage": "The image id 'ami-4edb0327' does not exist. Launching EC2 instance failed."
        }
   ]
}
```

The following tables list the types of error messages and provide links to the troubleshooting resources that you can use to troubleshoot issues\.


**Launch issues**  

| Issue | Error message | 
| --- | --- | 
| Availability Zone | [The requested Availability Zone is no longer supported\. Please retry your request\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-5) | 
| Block device mapping | [Invalid device name upload\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-8) | 
| Block device mapping | [Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-9) | 
| Block device mapping | [EBS block device mappings not supported for instance\-store AMIs\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-10) | 
| Instance configuration | [The requested configuration is currently not supported\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-3) | 
| Instance type and Availability Zone | [Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-6) | 
| Insufficient instance capacity in Availability Zone | [We currently do not have sufficient <instance type> capacity in the Availability Zone you requested\.\.\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-capacity-1) | 
| Insufficient instance capacity for a Spot request | [There is no Spot capacity available that matches your request\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-capacity-2) | 
| Key pair | [The key pair <key pair associated with your EC2 instance> does not exist\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-2) | 
| Placement group | [Placement groups may not be used with instances of type 'm1\.large'\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-11) | 
| Quota limits | [<number of instances> instance\(s\) are already running\. Launching EC2 instance failed\. ](ts-as-instancelaunchfailure.md#ts-as-capacity-3) | 
| Security group | [The security group <name of the security group> does not exist\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-1) | 
| Service\-linked role | [Client\.InternalError: Client error on launch\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-12) | 
| Spot price too low | [Your Spot request price of 0\.015 is lower than the minimum required Spot request fulfillment price of 0\.0735\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-7) | 


**AMI issues**  

| Issue | Error message | 
| --- | --- | 
| AMI ID | [The AMI ID <ID of your AMI> does not exist\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-1) | 
| AMI ID | [AMI <AMI ID> is pending, and cannot be run\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-2) | 
| AMI ID | [Value \(<ami ID>\) for parameter virtualName is invalid\.](ts-as-ami.md#ts-as-ami-4) | 
| Architecture mismatch | [The requested instance type's architecture \(i386\) does not match the architecture in the manifest for ami\-6622f00f \(x86\_64\)\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-5) | 


**Load balancer issues**  

| Issue | Error message | 
| --- | --- | 
| Cannot find load balancer | [Cannot find Load Balancer <your launch environment>\. Validating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-1) | 
| Instances in VPC | [EC2 instance <instance ID> is not in VPC\. Updating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-3) | 
| No active load balancer | [There is no ACTIVE Load Balancer named <load balancer name>\. Updating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-2) | 
| Security token | [The security token included in the request is invalid\. Validating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-4) | 