# Creating a launch template for an Auto Scaling group<a name="create-launch-template"></a>

Before you can create an Auto Scaling group using a launch template, you must create a launch template that includes the parameters required to launch an EC2 instance, such as the ID of the Amazon Machine Image \(AMI\) and an instance type\.

A launch template provides full functionality for Amazon EC2 Auto Scaling and also newer features of Amazon EC2 such as the current generation of EBS provisioned IOPS volumes \(io2\), EBS volume tagging, T2 Unlimited instances, elastic inference, and Dedicated Hosts\.

The following procedure works for creating a new launch template\. After you create your launch template, you can create the Auto Scaling group by following the instructions in [Creating an Auto Scaling group using a launch template](create-asg-launch-template.md)\.

**Topics**
+ [Creating your launch template \(console\)](#create-launch-template-for-auto-scaling)
+ [Creating a launch template from an existing instance \(console\)](#create-launch-template-from-instance)
+ [Creating a launch template \(AWS CLI\)](#create-launch-template-aws-cli)
+ [Limitations](#create-launch-template-limitations)

## Creating your launch template \(console\)<a name="create-launch-template-for-auto-scaling"></a>

Follow these steps to configure your launch template for the following: 
+ Specify the Amazon machine image \(AMI\) from which to launch the instances\.
+ Choose an instance type that is compatible with the AMI you've specified\. 
+ Specify the key pair to use when connecting to instances, for example, using SSH\.
+ Add one or more security groups to allow relevant access to the instances from an external network\.
+ Specify whether to attach additional EBS volumes or instance store volumes to each instance\. 
+ Add custom tags \(key\-value pairs\) to the instances and volumes\.

**To create a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **INSTANCES**, choose **Launch Templates**\.

1. Choose **Create launch template**\. Enter a name and provide a description for the initial version of the launch template\.

1. Under **Auto Scaling guidance**, select the check box to have Amazon EC2 provide guidance to help create a template to use with Amazon EC2 Auto Scaling\. 

1. Under **Launch template contents**, fill out each required field and any optional fields to use as your instance launch specification\.

   1. **Amazon machine image \(AMI\)**: Choose the ID of the AMI from which to launch the instances\. You can search through all available AMIs, or from the **Quick Start** list, select from one of the commonly used AMIs in the list\. If you don't see the AMI that you need, you can [find a suitable AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html), make note of its ID, and specify it as a custom value\.

   1. **Instance type**: Choose the [instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)\.

   1. \(Optional\) **Key pair \(login\)**: Specify a [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)\.

1. \(Optional\) Under **Network settings**, do the following:

   1. **Networking platform**: Choose whether to launch instances into a VPC or EC2\-Classic, if applicable\. However, the network type and Availability Zone settings of the launch template are ignored for Amazon EC2 Auto Scaling in favor of the settings of the Auto Scaling group\. 

   1. **Security groups**: Choose one or more [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html), or leave blank to configure one or more security groups as part of the network interface\. Each security group must be configured for the VPC that your Auto Scaling group will launch instances into\. If you're using EC2\-Classic, you must use security groups created specifically for EC2\-Classic\. 

      If you don't specify any security groups in your launch template, Amazon EC2 uses the [default security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#DefaultSecurityGroup)\. By default, this security group doesn't allow inbound traffic from external networks\.

1. \(Optional\) For **Storage \(Volumes\)**, specify volumes to attach to the instances in addition to the volumes specified by the AMI \(**Volume 1 \(AMI Root\)**\)\. To add a new volume, choose **Add new volume**\.

   1. **Volume type**: The type of volume depends on the instance type that you've chosen\. Each instance type has an associated root device volume, either an Amazon EBS volume or an instance store volume\. For more information, see [Amazon EC2 instance store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) and [Amazon EBS volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumes.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **Device name**: Specify a device name for the volume\.

   1. **Snapshot**: Enter the ID of the snapshot from which to create the volume\.

   1. **Size \(GiB\)**: For Amazon EBS\-backed volumes, specify a storage size\. If you're creating the volume from a snapshot and don't specify a volume size, the default is the snapshot size\.

   1. **Volume type**: For Amazon EBS volumes, choose the [volume type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)\.

   1. **IOPS**: With a Provisioned IOPS SSD volume, enter the maximum number of input/output operations per second \(IOPS\) that the volume should support\.

   1. **Delete on termination**: For Amazon EBS volumes, choose whether to delete the volume when the associated instance is terminated\. 

   1. **Encrypted**: Choose **Yes** to change the encryption state of an Amazon EBS volume\. The default effect of setting this parameter varies with the choice of volume source, as described in the table below\. You must in all cases have permission to use the specified CMK\. For more information about specifying encrypted volumes, see [Amazon EBS encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Linux Instances*\.   
**Encryption outcomes**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html)

      \* If [encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default) is enabled, all newly created volumes \(whether or not the **Encrypted** parameter is set to **Yes**\) are encrypted using the default CMK\. Setting both the **Encrypted** and **Key** parameters allows you to specify a non\-default CMK\. 

   1. **Key**: If you chose **Yes** in the previous step, optionally enter the customer master key \(CMK\) you want to use when encrypting the volumes\. Enter any CMK that you previously created using the AWS Key Management Service\. You can paste the full ARN of any key that you have access to\. For more information, see the [AWS Key Management Service Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/) and the [Required CMK key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md) topic in this guide\. Note: Amazon EBS does not support asymmetric CMKs\. 
**Note**  
Providing a CMK without also setting the **Encrypted** parameter results in an error\. 

1. For **Instance tags**, specify tags by providing key and value combinations\. You can tag the instances, the volumes, or both\.

1. To change the default network interface, see [Changing the default network interface](#change-network-interface)\. Skip this step if you want to keep the default network interface \(the primary network interface\)\.

1. To configure advanced settings, see [Configuring advanced settings for your launch template](#advanced-settings-for-your-launch-template)\. Otherwise, choose **Create launch template**\. 

1. To create an Auto Scaling group, choose **Create Auto Scaling group** from the confirmation page\.

### Changing the default network interface<a name="change-network-interface"></a>

This section shows you how to change the default network interface\. This allows you to define, for example, whether you want to assign a public IP address to each instance instead of defaulting to the auto\-assign public IP setting on the subnet\.

**Considerations and limitations**

When specifying a network interface, keep in mind the following considerations and limitations:
+ You must configure the security group as part of the network interface, and not in the **Security groups** section of the template\. You cannot specify security groups in both places\.
+ You cannot assign specific private IP addresses to your Auto Scaling instances\. When an instance launches, a private address is allocated from the CIDR range of the subnet in which the instance is launched\. For more information on specifying CIDR ranges for your VPC or subnet, see the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\.
+ You can launch only one instance if you specify an existing network interface ID\. For this to work, you must use the AWS CLI or an SDK to create the Auto Scaling group\. When you create the group, you must specify the Availability Zone, but not the subnet ID\. Also, you can specify an existing network interface only if it has a device index of 0\. 
+ You cannot auto\-assign a public IP address if you specify more than one network interface\. You also cannot specify duplicate device indexes across network interfaces\. Note that both the primary and secondary network interfaces will reside in the same subnet\.

**To change the default network interface**

1. Under **Network interfaces**, choose **Add network interface**\.

1. Specify the primary network interface, paying attention to the following fields:

   1. **Device index**: Specify the device index\. Enter `0` for the primary network interface \(eth0\)\. 

   1. **Network interface**: Leave blank to create a new network interface when an instance is launched, or enter the ID of an existing network interface\. If you specify an ID, this limits your Auto Scaling group to one instance\. 

   1. **Description**: Enter a descriptive name\.

   1. **Subnet**: While you can choose to specify a subnet, it is ignored for Amazon EC2 Auto Scaling in favor of the settings of the Auto Scaling group\. 

   1. **Auto\-assign public IP**: Choose whether to automatically assign a [public IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#public-ip-addresses) to the network interface with the device index of 0\. This setting takes precedence over settings that you configure for the subnets\. If you do not set a value, the default is to use the auto\-assign public IP settings of the subnets that your instances are launched into\. 

   1. **Security groups**: Choose one or more [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)\. Each security group must be configured for the VPC that your Auto Scaling group will launch instances into\.

   1. **Delete on termination**: Choose whether the network interface is deleted when the Auto Scaling group scales in and terminates the instance to which the network interface is attached\. 

   1. **Elastic Fabric Adapter**: Indicates whether the network interface is an Elastic Fabric Adapter\. For more information, see [Elastic Fabric Adapter](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

   1. **Network card index**: Attaches the network interface to a specific network card when using an instance type that supports multiple network cards\. The primary network interface \(eth0\) must be assigned to network card index 0\. Defaults to 0 if not specified\. For more information, see [Network cards](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#network-cards) in the *Amazon EC2 User Guide for Linux Instances*\. 

1. To add a secondary network interface, choose **Add network interface**\.

### Configuring advanced settings for your launch template<a name="advanced-settings-for-your-launch-template"></a>

You can define any additional capabilities that your Auto Scaling instances need\. For example, you can choose an IAM role that your application can use when it accesses other AWS resources or specify the instance user data that can be used to perform common automated configuration tasks after an instance starts\. 

The following steps discuss the most useful settings to pay attention to\. For more information about any of the settings under **Advanced details**, see [Creating a launch template](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#create-launch-template) in the *Amazon EC2 User Guide for Linux Instances*\.

**To configure advanced settings**

1. For **Advanced details**, expand the section to view the fields\.

1. For **Purchasing option**, you can choose **Request Spot Instances** to request Spot Instances at the Spot price, capped at the On\-Demand price, and choose **Customize** to change the default Spot Instance settings\. For an Auto Scaling group, you must specify a one\-time request with no end date \(the default\)\. For more information, see [Requesting Spot Instances for fault\-tolerant and flexible applications](asg-launch-spot-instances.md)\. 
**Note**  
If you leave this setting disabled, you can request Spot Instances later in your Auto Scaling group\. This also gives you the option of specifying multiple instance types\. That way, if the Amazon EC2 Spot service needs to reclaim your Spot Instances, we can launch replacement instances from another Spot pool\. For more information, see [Auto Scaling groups with multiple instance types and purchase options](asg-purchase-options.md)\.

1. For **IAM instance profile**, you can specify an AWS Identity and Access Management \(IAM\) instance profile to associate with the instances\. When you choose an instance profile, you associate the corresponding IAM role with the EC2 instances\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

1. For **Termination protection**, choose whether to protect instances from accidental termination\. When you enable termination protection, it provides additional termination protection, but it does not protect from Amazon EC2 Auto Scaling initiated termination\. To control whether an Auto Scaling group can terminate a particular instance, use [Instance scale\-in protection](as-instance-termination.md#instance-protection)\.

1. For **Detailed CloudWatch monitoring**, choose whether to enable the instances to publish metric data at 1\-minute intervals to Amazon CloudWatch\. Additional charges apply\. For more information, see [Configuring monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\.

1. For **T2/T3 Unlimited**, choose whether to enable applications to burst beyond the baseline for as long as needed\. This field is only valid for T2, T3, and T3a instances\. Additional charges may apply\. For more information, see [Using an Auto Scaling group to launch a burstable performance instance as Unlimited](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-how-to.html#burstable-performance-instances-auto-scaling-grp) in the *Amazon EC2 User Guide for Linux Instances*\.

1. For **Placement group name**, you can specify a placement group in which to launch the instances\. Not all instance types can be launched in a placement group\. If you configure an Auto Scaling group using a CLI command that specifies a different placement group, the setting is ignored in favor of the one specified for the Auto Scaling group\.

1. For **Capacity Reservation**, you can specify whether to launch the instances into shared capacity, any `open` Capacity Reservation, a specific Capacity Reservation, or a Capacity Reservation group\. For more information, see [Launching instances into an existing capacity reservation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/capacity-reservations-using.html#capacity-reservations-launch) in the *Amazon EC2 User Guide for Linux Instances*\.

1. For **Tenancy**, you can choose to run your instances on shared hardware \(**Shared**\), on dedicated hardware \(**Dedicated**\), or when using a host resource group, on Dedicated Hosts \(**Dedicated host**\)\. Additional charges may apply\. 

   If you chose **Dedicated Hosts**, complete the following information:

   1. For **Tenancy host resource group**, you can specify a host resource group for a BYOL AMI to use on Dedicated Hosts\. You do not have to have already allocated Dedicated Hosts in your account before you use this feature\. Your instances will automatically launch onto Dedicated Hosts regardless\. Note that an AMI based on a license configuration association can be mapped to only one host resource group at a time\. For more information, see [Host resource groups](https://docs.aws.amazon.com/license-manager/latest/userguide/host-resource-groups.html) in the *AWS License Manager User Guide*\. 

1. For **License configurations**, specify the license configuration to use\. You can launch instances against the specified license configuration to track your license usage\. For more information, see [Create a license configuration](https://docs.aws.amazon.com/license-manager/latest/userguide/create-license-configuration.html) in the *License Manager User Guide*\.

1. To configure instance metadata options for all of the instances that are associated with this version of the launch template, do the following:

   1. For **Metadata accessible**, choose whether to enable or disable access to the HTTP endpoint of the instance metadata service\. By default, the HTTP endpoint is enabled\. If you choose to disable the endpoint, access to your instance metadata is turned off\. You can specify the condition to require IMDSv2 only when the HTTP endpoint is enabled\. 

   1. For **Metadata version**, you can choose to require the use of Instance Metadata Service Version 2 \(IMDSv2\) when requesting instance metadata\. If you do not specify a value, the default is to support both IMDSv1 and IMDSv2\.

   1. For **Metadata token response hop limit**, you can set the allowable number of network hops for the metadata token\. If you do not specify a value, the default is 1\.

   For more information, see [Configuring the instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) in the *Amazon EC2 User Guide for Linux Instances*\.

1. For **User data**, you can specify user data to configure an instance during launch, or to run a configuration script after the instance starts\.

1. Choose **Create launch template**\.

1. To create an Auto Scaling group, choose **Create Auto Scaling group** from the confirmation page\.

## Creating a launch template from an existing instance \(console\)<a name="create-launch-template-from-instance"></a>

**To create a launch template from an existing instance**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **INSTANCES**, choose **Instances**\.

1. Select the instance and choose **Actions**, **Image and templates**, **Create template from instance**\.

1. Provide a name and description\. 

1. Under **Auto Scaling guidance**, select the check box\. 

1. Adjust any settings as required, and choose **Create launch template**\. 

1. To create an Auto Scaling group, choose **Create Auto Scaling group** from the confirmation page\.

## Creating a launch template \(AWS CLI\)<a name="create-launch-template-aws-cli"></a>

**To create a launch template using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2LaunchTemplate.html](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2LaunchTemplate.html) \(AWS Tools for Windows PowerShell\)

## Limitations<a name="create-launch-template-limitations"></a>
+ A launch template lets you configure a network type \(VPC or EC2\-Classic\), subnet, and Availability Zone\. However, these settings are ignored in favor of what is specified in the Auto Scaling group\. 
+ Because the subnet settings in your launch template are ignored in favor of what is specified in the Auto Scaling group, all of the network interfaces that are created for a given instance will be connected to the same subnet as the instance\. For other limitations on user\-defined network interfaces, see [Changing the default network interface](#change-network-interface)\.
+ A launch template lets you configure additional settings in your Auto Scaling group to launch multiple instance types and combine On\-Demand and Spot purchase options, as described in [Auto Scaling groups with multiple instance types and purchase options](asg-purchase-options.md)\. Launching instances with such a combination is not supported:
  + If you specify a Spot Instance request in the launch template
  + In EC2\-Classic
+ Support for Dedicated Hosts \(host tenancy\) is only available if you specify a host resource group\. You cannot target a specific host ID or use host placement affinity\.