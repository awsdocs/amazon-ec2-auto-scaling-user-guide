# Troubleshooting Amazon EC2 Auto Scaling: EC2 Instance launch failures<a name="ts-as-instancelaunchfailure"></a>

This page provides information about your EC2 instances that fail to launch, potential causes, and the steps you can take to resolve the issues\.

To retrieve an error message, see [Retrieving an error message](CHAP_Troubleshooting.md#RetrievingErrors)\.

When your EC2 instances fail to launch, you might get one or more of the following error messages:

**Topics**
+ [The requested configuration is currently not supported\.](#ts-as-instancelaunchfailure-3)
+ [The security group <name of the security group> does not exist\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-1)
+ [The key pair <key pair associated with your EC2 instance> does not exist\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-2)
+ [The requested Availability Zone is no longer supported\. Please retry your request\.\.\.](#ts-as-instancelaunchfailure-5)
+ [Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\.\.\.](#ts-as-instancelaunchfailure-6)
+ [Your Spot request price of 0\.015 is lower than the minimum required Spot request fulfillment price of 0\.0735\.\.\.](#ts-as-instancelaunchfailure-7)
+ [Invalid device name upload\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-8)
+ [Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\.\.\.](#ts-as-instancelaunchfailure-9)
+ [EBS block device mappings not supported for instance\-store AMIs\.](#ts-as-instancelaunchfailure-10)
+ [Placement groups may not be used with instances of type 'm1\.large'\. Launching EC2 instance failed\.](#ts-as-instancelaunchfailure-11)
+ [Client\.InternalError: Client error on launch\.](#ts-as-instancelaunchfailure-12)
+ [We currently do not have sufficient <instance type> capacity in the Availability Zone you requested\.\.\. Launching EC2 instance failed\.](#ts-as-capacity-1)
+ [There is no Spot capacity available that matches your request\. Launching EC2 instance failed\.](#ts-as-capacity-2)
+ [<number of instances> instance\(s\) are already running\. Launching EC2 instance failed\.](#ts-as-capacity-3)

## The requested configuration is currently not supported\.<a name="ts-as-instancelaunchfailure-3"></a>
+ **Cause**: Some options in your launch template or launch configuration might not be compatible with the instance type, or the instance configuration might not be supported in your requested AWS Region or Availability Zones\.
+ **Solution**: 

  Try a different instance configuration\. To search for an instance type that meets your requirements, see [Finding an Amazon EC2 instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-discovery.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

  For further guidance to resolve this issue, check the following:
  + Ensure that you have chosen an AMI that is supported by your instance type\. For example, if the instance type uses an Arm\-based AWS Graviton processor instead of an Intel Xeon processor, you need an Arm\-compatible AMI\.
  + Test that the instance type is available in your requested Region and Availability Zones\. The newest generation instance types might not yet be available in a given Region or Availability Zone\. The older generation instance types might not be available in newer Regions and Availability Zones\. To search for instance types offered by location \(Region or Availability Zone\), use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instance-type-offerings.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instance-type-offerings.html) command\. For more information, see [Finding an Amazon EC2 instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-discovery.html) in the *Amazon EC2 User Guide for Linux Instances*\. 
  + If you use Dedicated Instances or Dedicated Hosts, ensure that you have chosen an instance type that's supported as a Dedicated Instance or Dedicated Host\. 

## The security group <name of the security group> does not exist\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-1"></a>
+ **Cause**: The security group specified in your launch template or launch configuration might have been deleted\. 
+ **Solution**: 

  1. Use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html) command to get the list of the security groups associated with your account\.

  1. From the list, select the security groups to use\. To create a security group instead, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html) command\.

  1. Create a new launch template or launch configuration\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## The key pair <key pair associated with your EC2 instance> does not exist\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-2"></a>
+ **Cause**: The key pair that was used when launching the instance might have been deleted\.
+ **Solution**: 

  1. Use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-key-pairs.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-key-pairs.html) command to get the list of the key pairs available to you\.

  1. From the list, select the key pair to use\. To create a key pair instead, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-key-pair.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-key-pair.html) command\.

  1. Create a new launch template or launch configuration\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## The requested Availability Zone is no longer supported\. Please retry your request\.\.\.<a name="ts-as-instancelaunchfailure-5"></a>
+ **Error message**: The requested Availability Zone is no longer supported\. Please retry your request by not specifying an Availability Zone or choosing <list of available Availability Zones>\. Launching EC2 instance failed\.
+ **Cause**: The Availability Zone associated with your Auto Scaling group might not be currently available\.
+ **Solution**: Update your Auto Scaling group with the recommendations in the error message\. 

## Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\.\.\.<a name="ts-as-instancelaunchfailure-6"></a>
+ **Error message**: Your requested instance type \(<instance type>\) is not supported in your requested Availability Zone \(<instance Availability Zone>\)\. Please retry your request by not specifying an Availability Zone or choosing <list of Availability Zones that supports the instance type>\. Launching EC2 instance failed\.
+ **Cause**: The instance type you chose might not be currently available in the Availability Zones specified in your Auto Scaling group\. 
+ **Solution**: Update your Auto Scaling group with the recommendations in the error message\.

## Your Spot request price of 0\.015 is lower than the minimum required Spot request fulfillment price of 0\.0735\.\.\.<a name="ts-as-instancelaunchfailure-7"></a>
+ **Cause**: The Spot maximum price in your request is lower than the Spot price for the instance type that you selected\. 
+ **Solution**: Submit a new request with a higher Spot maximum price \(possibly the On\-Demand price\)\. Previously, the Spot price you paid was based on bidding\. Today, you pay the current Spot price\. By setting your maximum price higher, it gives the Amazon EC2 Spot service a better chance of launching and maintaining your required amount of capacity\.

## Invalid device name upload\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-8"></a>
+ **Cause**: The block device mappings in your launch template or launch configuration might contain block device names that are not available or currently not supported\. 
+ **Solution**: 

  1. Use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-volumes.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-volumes.html) command to see how the volumes are exposed to the instance\.

  1. Create a new launch template or launch configuration using the device name listed in the volume description\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\.\.\.<a name="ts-as-instancelaunchfailure-9"></a>
+ **Error message**: Value \(<name associated with the instance storage device>\) for parameter virtualName is invalid\. Expected format: 'ephemeralNUMBER'\. Launching EC2 instance failed\.
+ **Cause**: The format specified for the virtual name associated with the block device is incorrect\. 
+ **Solution**:

  1. Create a new launch template or launch configuration by specifying the device name in the `virtualName` parameter\. For information about the device name format, see [Device naming on Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/device_naming.html) in the *Amazon EC2 User Guide for Linux Instances*\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## EBS block device mappings not supported for instance\-store AMIs\.<a name="ts-as-instancelaunchfailure-10"></a>
+ **Cause**: The block device mappings specified in the launch template or launch configuration are not supported on your instance\. 
+ **Solution**:

  1. Create a new launch template or launch configuration with block device mappings supported by your instance type\. For more information, see [Block device mapping](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/block-device-mapping-concepts.html) in the *Amazon EC2 User Guide for Linux Instances*\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## Placement groups may not be used with instances of type 'm1\.large'\. Launching EC2 instance failed\.<a name="ts-as-instancelaunchfailure-11"></a>
+ **Cause**: Your cluster placement group contains an invalid instance type\. 
+ **Solution**: 

  1. For information about valid instance types supported by the placement groups, see [Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html) in the *Amazon EC2 User Guide for Linux Instances*\.

  1. Follow the instructions detailed in the [Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html) to create a new placement group\.

  1.  Alternatively, create a new launch template or launch configuration with the supported instance type\. 

  1. Update your Auto Scaling group with a new placement group, launch template, or launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## Client\.InternalError: Client error on launch\.<a name="ts-as-instancelaunchfailure-12"></a>
+ **Cause**: This error can be caused when an Auto Scaling group attempts to launch an instance that has an encrypted EBS volume, but the service\-linked role does not have access to the customer managed CMK used to encrypt it\. For more information, see [Required CMK key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md)\.
+ **Solution**: Additional setup is required to allow the Auto Scaling group to launch instances\. The following table summarizes the steps for resolving the error\. For more information, see [https://forums\.aws\.amazon\.com/thread\.jspa?threadID=277523](https://forums.aws.amazon.com/thread.jspa?threadID=277523)\.


| Scenario | Next steps | 
| --- | --- | 
|  **Scenario 1:**  CMK and Auto Scaling group are in the same AWS account  |  Allow the service\-linked role to use the CMK as follows: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ts-as-instancelaunchfailure.html)  | 
|  **Scenario 2:** CMK and Auto Scaling group are in different AWS accounts  |  There are two possible solutions: Solution 1: Use a CMK in the same AWS account as the Auto Scaling group [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ts-as-instancelaunchfailure.html) Solution 2: Continue to use the CMK in a different AWS account from the Auto Scaling group [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ts-as-instancelaunchfailure.html)  | 

## We currently do not have sufficient <instance type> capacity in the Availability Zone you requested\.\.\. Launching EC2 instance failed\.<a name="ts-as-capacity-1"></a>
+ **Error message**: We currently do not have sufficient <instance type> capacity in the Availability Zone you requested \(<requested Availability Zone>\)\. Our system will be working on provisioning additional capacity\. You can currently get <instance type> capacity by not specifying an Availability Zone in your request or choosing <list of Availability Zones that currently supports the instance type>\. Launching EC2 instance failed\.
+ **Cause**: At this time, Amazon EC2 cannot support your instance type in your requested Availability Zone\. 
+ **Solution**: 

  To resolve the issue, try the following:
  + Wait a few minutes and then submit your request again; capacity can shift frequently\. 
  + Submit a new request following the recommendations in the error message\.
  + Submit a new request with a reduced number of instances \(which you can increase at a later stage\)\.

## There is no Spot capacity available that matches your request\. Launching EC2 instance failed\.<a name="ts-as-capacity-2"></a>
+ **Cause**: At this time, there isn't enough spare capacity to fulfill your request for Spot Instances\. 
+ **Solution**: 

  To resolve the issue, try the following:
  + Wait a few minutes; capacity can shift frequently\. The Spot request continues to automatically make the launch request until capacity becomes available\. When capacity becomes available, the Amazon EC2 Spot service fulfills the Spot request\.
  + Follow the best practice of using a diverse set of instance types so that you are not reliant on one specific instance type\. For more information, including a list of best practices for using Spot Instances successfully, see [Auto Scaling groups with multiple instance types and purchase options](asg-purchase-options.md)\. 
  + Submit a new request with a reduced number of instances \(which you can increase at a later stage\)\.

## <number of instances> instance\(s\) are already running\. Launching EC2 instance failed\.<a name="ts-as-capacity-3"></a>
+ **Cause**: You have reached the limit on the number of instances that you can launch in a Region\. When you create your AWS account, we set default limits on the number of instances you can run on a per\-Region basis\. 
+ **Solution**:

  To resolve the issue, try the following:
  + If your current limits aren't adequate for your needs, you can request a quota increase on a per\-Region basis\. For more information, see [Amazon EC2 service quotas](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-resource-limits.html) in the *Amazon EC2 User Guide for Linux Instances*\.
  + Submit a new request with a reduced number of instances \(which you can increase at a later stage\)\.