# Troubleshooting Amazon EC2 Auto Scaling: EC2 Instance launch failures<a name="ts-as-instancelaunchfailure"></a>

This page provides information about your EC2 instances that fail to launch, potential causes, and the steps you can take to resolve the issues\.

To retrieve an error message, see [Retrieving an error message](CHAP_Troubleshooting.md#RetrievingErrors)\.

When your EC2 instances fail to launch, you might get one or more of the following error messages:

**Topics**
+ [The security group <name of the security group> does not exist\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-1)
+ [The key pair <key pair associated with your EC2 instance> does not exist\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-2)
+ [The requested configuration is currently not supported\.](#ts-as-instancelaunchfailure-3)
+ [AutoScalingGroup <Auto Scaling group name> not found\.](#ts-as-instancelaunchfailure-4)
+ [The requested Availability Zone is no longer supported\. Please retry your request\.\.\.](#ts-as-instancelaunchfailure-5)
+ [Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\.\.\.](#ts-as-instancelaunchfailure-6)
+ [You are not subscribed to this service\. Please see https://aws\.amazon\.com/\.](#ts-as-instancelaunchfailure-7)
+ [Invalid device name upload\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-8)
+ [Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\.\.\.](#ts-as-instancelaunchfailure-9)
+ [EBS block device mappings not supported for instance\-store AMIs\.](#ts-as-instancelaunchfailure-10)
+ [Placement groups may not be used with instances of type 'm1\.large'\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-11)
+ [Client\.InternalError: Client error on launch\.](#ts-as-instancelaunchfailure-12)

## The security group <name of the security group> does not exist\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-1"></a>
+ **Cause**: The security group specified in your launch configuration might have been deleted\. 
+ **Solution**: 

  1. Use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html) command to get the list of the security groups associated with your account\.

  1. From the list, select the security groups to use\. To create a security group instead, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html) command\.

  1. Create a new launch configuration\.

  1. Update your Auto Scaling group with the new launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## The key pair <key pair associated with your EC2 instance> does not exist\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-2"></a>
+ **Cause**: The key pair that was used when launching the instance might have been deleted\.
+ **Solution**: 

  1. Use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-key-pairs.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-key-pairs.html) command to get the list of the key pairs available to you\.

  1. From the list, select the key pair to use\. To create a key pair instead, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-key-pair.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-key-pair.html) command\.

  1. Create a new launch configuration\.

  1. Update your Auto Scaling group with the new launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## The requested configuration is currently not supported\.<a name="ts-as-instancelaunchfailure-3"></a>
+ **Cause**: Some options in your launch configuration might not be currently supported\.
+ **Solution**: 

  1. Create a new launch configuration\.

  1. Update your Auto Scaling group with the new launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## AutoScalingGroup <Auto Scaling group name> not found\.<a name="ts-as-instancelaunchfailure-4"></a>
+ **Cause**: The Auto Scaling group might have been deleted\.
+ **Solution**: Create a new Auto Scaling group\.

## The requested Availability Zone is no longer supported\. Please retry your request\.\.\.<a name="ts-as-instancelaunchfailure-5"></a>
+ **Error message**: The requested Availability Zone is no longer supported\. Please retry your request by not specifying an Availability Zone or choosing <list of available Availability Zones>\. Launching EC2 instance failed\.
+ **Cause**: The Availability Zone associated with your Auto Scaling group might not be currently available\.
+ **Solution**: Update your Auto Scaling group with the recommendations in the error message\. 

## Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\.\.\.<a name="ts-as-instancelaunchfailure-6"></a>
+ **Error message**: Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\. Please retry your request by not specifying an Availability Zone or choosing <list of Availability Zones that supports the instance type>\. Launching EC2 instance failed\.
+ **Cause**: The instance type associated with your launch configuration might not be currently available in the Availability Zones specified in your Auto Scaling group\. 
+ **Solution**: Update your Auto Scaling group with the recommendations in the error message\.

## You are not subscribed to this service\. Please see https://aws\.amazon\.com/\.<a name="ts-as-instancelaunchfailure-7"></a>
+ **Cause**: Your AWS account might have expired\. 
+ **Solution**: Go to https://aws\.amazon\.com/ and choose **Sign Up Now** to open a new account\. 

## Invalid device name upload\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-8"></a>
+ **Cause**: The block device mappings in your launch configuration might contain block device names that are not available or currently not supported\. 
+ **Solution**: 

  1. Use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-volumes.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-volumes.html) command to see how the volumes are exposed to the instance\.

  1. Create a new launch configuration using the device name listed in the volume description\.

  1. Update your Auto Scaling group with the new launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\.\.\.<a name="ts-as-instancelaunchfailure-9"></a>
+ **Error message**: Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\. Expected format: 'ephemeralNUMBER'\. Launching EC2 instance failed\.
+ **Cause**: The format specified for the virtual name associated with the block device is incorrect\. 
+ **Solution**:

  1. Create a new launch configuration by specifying the device name in the `virtualName` parameter\. For information about the device name format, see [Device naming on Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/device_naming.html) in the *Amazon EC2 User Guide for Linux Instances*\.

  1. Update your Auto Scaling group with the new launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## EBS block device mappings not supported for instance\-store AMIs\.<a name="ts-as-instancelaunchfailure-10"></a>
+ **Cause**: The block device mappings specified in the launch configuration are not supported on your instance\. 
+ **Solution**:

  1. Create a new launch configuration with block device mappings supported by your instance type\. For more information, see [Block device mapping](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/block-device-mapping-concepts.html) in the *Amazon EC2 User Guide for Linux Instances*\.

  1. Update your Auto Scaling group with the new launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## Placement groups may not be used with instances of type 'm1\.large'\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-11"></a>
+ **Cause**: Your cluster placement group contains an invalid instance type\. 
+ **Solution**: 

  1. For information about valid instance types supported by the placement groups, see [Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html) in the *Amazon EC2 User Guide for Linux Instances*\.

  1. Follow the instructions detailed in the [Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html) to create a new placement group\.

  1.  Alternatively, create a new launch configuration with the supported instance type\. 

  1. Update your Auto Scaling group with a new placement group or launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## Client\.InternalError: Client error on launch\.<a name="ts-as-instancelaunchfailure-12"></a>
+ **Cause**: This error can be caused when an Auto Scaling group attempts to launch an instance that has an encrypted EBS volume, but the service\-linked role does not have access to the customer managed CMK used to encrypt it\. For more information, see [Required CMK key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md)\.
+ **Solution**: Additional setup is required to allow the Auto Scaling group to launch instances\. The following table summarizes the steps for resolving the error\. For more information, see [https://forums\.aws\.amazon\.com/thread\.jspa?threadID=277523](https://forums.aws.amazon.com/thread.jspa?threadID=277523)\.


| Scenario | Next steps | 
| --- | --- | 
|  **Scenario 1:**  CMK and Auto Scaling group are in the same AWS account  |  Allow the service\-linked role to use the CMK as follows: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ts-as-instancelaunchfailure.html)  | 
|  **Scenario 2:** CMK and Auto Scaling group are in different AWS accounts  |  There are two possible solutions: Solution 1: Use a CMK in the same AWS account as the Auto Scaling group [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ts-as-instancelaunchfailure.html) Solution 2: Continue to use the CMK in a different AWS account from the Auto Scaling group [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ts-as-instancelaunchfailure.html)  | 