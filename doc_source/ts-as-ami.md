# Troubleshoot Amazon EC2 Auto Scaling: AMI issues<a name="ts-as-ami"></a>

This page provides information about the issues associated with your AMIs, potential causes, and the steps you can take to resolve the issues\.

To retrieve an error message, see [Retrieve an error message from scaling activities](CHAP_Troubleshooting.md#RetrievingErrors)\.

When your EC2 instances fail to launch due to issues with your AMI, you might get one or more of the following error messages\.

**Topics**
+ [The AMI ID <ID of your AMI> does not exist\. Launching EC2 instance failed\.](#ts-as-ami-1)
+ [AMI <AMI ID> is pending, and cannot be run\. Launching EC2 instance failed\.](#ts-as-ami-2)
+ [Value \(<ami ID>\) for parameter virtualName is invalid\.](#ts-as-ami-4)
+ [The requested instance type's architecture \(i386\) does not match the architecture in the manifest for ami\-6622f00f \(x86\_64\)\. Launching EC2 instance failed\.](#ts-as-ami-5)

## The AMI ID <ID of your AMI> does not exist\. Launching EC2 instance failed\.<a name="ts-as-ami-1"></a>
+ **Cause**: The AMI might have been deleted after creating the launch template or launch configuration\.
+ **Solution**: 

  1. Create a new launch template or launch configuration using a valid AMI\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## AMI <AMI ID> is pending, and cannot be run\. Launching EC2 instance failed\.<a name="ts-as-ami-2"></a>
+ **Cause**: You might have just created your AMI \(by taking a snapshot of a running instance or any other way\), and it might not be available yet\. 
+ **Solution**: You must wait for your AMI to be available and then create your launch template or launch configuration\. 

## Value \(<ami ID>\) for parameter virtualName is invalid\.<a name="ts-as-ami-4"></a>
+ **Cause**: Incorrect value\. The `virtualName` parameter refers to the virtual name associated with the device\. 
+ **Solution**:

  1. Create a new launch template or launch configuration by specifying the name of the virtual device of your instance for the `virtualName` parameter\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

## The requested instance type's architecture \(i386\) does not match the architecture in the manifest for ami\-6622f00f \(x86\_64\)\. Launching EC2 instance failed\.<a name="ts-as-ami-5"></a>
+ **Cause**: The architecture of the `InstanceType` mentioned in your launch template or launch configuration does not match the image architecture\. 
+ **Solution**:

  1. Create a new launch template or launch configuration using the AMI architecture that matches the architecture of the requested instance type\.

  1. Update your Auto Scaling group with the new launch template or launch configuration using the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.