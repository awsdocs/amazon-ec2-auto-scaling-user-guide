# Troubleshooting Amazon EC2 Auto Scaling<a name="CHAP_Troubleshooting"></a>

Amazon EC2 Auto Scaling provides specific and descriptive errors to help you troubleshoot issues\. You can find the error messages in the description of the scaling activities\.

**Topics**
+ [General troubleshooting issues](#troubleshooting-general)
+ [Retrieving an error message](#RetrievingErrors)
+ [Troubleshooting Amazon EC2 Auto Scaling: EC2 Instance launch failures](ts-as-instancelaunchfailure.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: AMI issues](ts-as-ami.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: Load balancer issues](ts-as-loadbalancer.md)
+ [Troubleshooting Auto Scaling: Capacity limits](ts-as-capacity.md)

## General troubleshooting issues<a name="troubleshooting-general"></a>

### Procedures in this guide do not match the console<a name="ts-new-console-procedures"></a>

The procedures in this guide are written to reflect the new console design for Amazon EC2 Auto Scaling\. If you are using the older version of the console, many of the concepts and basic procedures in this guide still apply\. To access help in the new console, choose the information icon\.

### Permissions required for a launch template are missing<a name="ts-launch-template-permissions"></a>

You can add a launch template to a new Auto Scaling group or to an existing Auto Scaling group\. If you are attempting to use a launch template, and you do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. For information about the permissions necessary to work with launch templates, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

### Permissions required for an IAM role are missing<a name="ts-instance-profile-permissions"></a>

To create or update an Auto Scaling group using a launch template that specifies an instance profile, you must have permission to pass IAM roles\. It's a permission that AWS also checks whenever you create a launch configuration that specifies an instance profile\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\. For further troubleshooting topics related to instance profiles, see [Troubleshooting Amazon EC2 and IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_iam-ec2.html) in the *IAM User Guide*\.

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


**EC2 instance launch failures**  

| Issue | Error message | 
| --- | --- | 
| Auto Scaling group | [AutoScalingGroup <Auto Scaling group name> not found\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-4) | 
| Availability Zone | [The requested Availability Zone is no longer supported\. Please retry your request\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-5) | 
| AWS account | [You are not subscribed to this service\. Please see https://aws\.amazon\.com/\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-7) | 
| Block device mapping | [Invalid device name upload\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-8) | 
| Block device mapping | [Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-9) | 
| Block device mapping | [EBS block device mappings not supported for instance\-store AMIs\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-10) | 
| Instance type and Availability Zone | [Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\.\.\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-6) | 
| Key pair | [The key pair <key pair associated with your EC2 instance> does not exist\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-2) | 
| Launch configuration | [The requested configuration is currently not supported\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-3) | 
| Placement group | [Placement groups may not be used with instances of type 'm1\.large'\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-11) | 
| Security group | [The security group <name of the security group> does not exist\. Launching EC2 instance failed\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-1) | 
| Service\-linked role | [Client\.InternalError: Client error on launch\.](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-12) | 


**AMI issues**  

| Issue | Error message | 
| --- | --- | 
| AMI ID | [The AMI ID <ID of your AMI> does not exist\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-1) | 
| AMI ID | [AMI <AMI ID> is pending, and cannot be run\. Launching EC2 instance failed\.](ts-as-ami.md#ts-as-ami-2) | 
| AMI ID | [Value \(<ami ID>\) for parameter virtualName is invalid\.](ts-as-ami.md#ts-as-ami-4) | 
| Architecture mismatch | [The requested instance type's architecture \(i386\) does not match the architecture in the manifest for ami\-6622f00f \(x86\_64\)\. Launching ec2 instance failed\.](ts-as-ami.md#ts-as-ami-5) | 


**Load balancer issues**  

| Issue | Error message | 
| --- | --- | 
| Cannot find load balancer | [Cannot find Load Balancer <your launch environment>\. Validating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-1) | 
| Instances in VPC | [EC2 instance <instance ID> is not in VPC\. Updating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-3) | 
| No active load balancer | [There is no ACTIVE Load Balancer named <load balancer name>\. Updating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-2) | 
| Security token | [The security token included in the request is invalid\. Validating load balancer configuration failed\.](ts-as-loadbalancer.md#ts-as-loadbalancer-4) | 


**Capacity limits**  

| Issue | Error message | 
| --- | --- | 
| Capacity limits | [<number of instances> instance\(s\) are already running\. Launching EC2 instance failed\.](ts-as-capacity.md#ts-as-capacity-2) | 
| Insufficient capacity in Availability Zone | [We currently do not have sufficient <instance type> capacity in the Availability Zone you requested \(<requested Availability Zone>\)\.\.\.](ts-as-capacity.md#ts-as-capacity-1) | 