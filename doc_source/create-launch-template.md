# Create a launch template for an Auto Scaling group<a name="create-launch-template"></a>

Before you can create an Auto Scaling group using a launch template, you must create a launch template that includes the parameters required to launch an EC2 instance, such as the ID of the Amazon Machine Image \(AMI\) and an instance type\.

A launch template provides full functionality for Amazon EC2 Auto Scaling and also newer features of Amazon EC2 such as the current generation of EBS Provisioned IOPS volumes \(io2\), EBS volume tagging, T2 Unlimited instances, Elastic Inference, and Dedicated Hosts\.

The following procedure works for creating a new launch template\. After you create your launch template, you can create the Auto Scaling group by following the instructions in these topics: 
+ [Create an Auto Scaling group using a launch template](create-asg-launch-template.md)
+ [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)
+ [Create an Auto Scaling group using attribute\-based instance type selection](create-asg-instance-type-requirements.md)

**Topics**
+ [Create your launch template \(console\)](#create-launch-template-for-auto-scaling)
+ [Create a launch template from an existing instance \(console\)](#create-launch-template-from-instance)
+ [Additional information](#create-launch-template-additional-info)
+ [Limitations](#create-launch-template-limitations)

## Create your launch template \(console\)<a name="create-launch-template-for-auto-scaling"></a>

Follow these steps to configure your launch template for the following: 
+ Specify the Amazon machine image \(AMI\) from which to launch the instances\.
+ Choose an instance type that is compatible with the AMI you've specified\. 
+ Specify the key pair to use when connecting to instances, for example, using SSH\.
+ Add one or more security groups to allow relevant access to the instances from an external network\.
+ Specify whether to attach additional EBS volumes or instance store volumes to each instance\. 
+ Add custom tags \(key\-value pairs\) to the instances and volumes\.

**To create a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Instances**, choose **Launch Templates**\.

1. Choose **Create launch template**\. Enter a name and provide a description for the initial version of the launch template\.

1. Under **Auto Scaling guidance**, select the check box to have Amazon EC2 provide guidance to help create a template to use with Amazon EC2 Auto Scaling\. 

1. Under **Launch template contents**, fill out each required field and any optional fields as needed\.

   1. **Application and OS Images \(Amazon Machine Image\)**: Choose the ID of the AMI from which to launch the instances\. You can search through all available AMIs, or from the **Recent** or **Quick Start** list, select an AMI\. If you don't see the AMI that you need, you can choose **Browser more AMIs** to browse the full AMI catalog\.

   1. For **Instance type**, choose a single instance type that is compatible with the AMI\. To configure advanced options, choose **Advanced**\.

      Use advanced options to support the use of multiple instance types when using attribute\-based instance type selection\. For more information, see [Create an Auto Scaling group using attribute\-based instance type selection](create-asg-instance-type-requirements.md)\. Manually choosing instance types is not supported when creating a launch template for an Auto Scaling group\.

      To use attribute\-based instance type selection, choose **Specify instance type attributes**, and specify the following options:
      + **Number of vCPUs**: Enter the minimum and maximum number of vCPUs\. To indicate no limits, enter a minimum of 0, and leave the maximum blank\. 
      + **Amount of memory \(MiB\)**: Enter the minimum and maximum amount of memory, in MiB\. To indicate no limits, enter a minimum of 0, and leave the maximum blank\. 
      + Expand **Optional instance type attributes** and choose **Add attribute** to further limit the types of instances that can be used to fulfill your desired capacity\. For information about each attribute, see [InstanceRequirementsRequest](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_InstanceRequirementsRequest.html) in the *Amazon EC2 API Reference*\. 
      + **Resulting instance types**: You can view the instance types that match the specified compute requirements, such as vCPUs, memory, and storage\.
      + \(Optional\) To exclude instance types, choose **Add attribute**, and from the **Attribute** list, choose **Excluded instance types**\. From the **Attribute value** list, select the instance types to exclude\.

   1. \(Optional\) **Key pair \(login\)**: Specify a [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)\.

   1. \(Optional\) **Network settings**: Choose one or more [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html), or leave blank and configure one or more security groups as part of the network interface\. Each security group must be configured for the VPC that your Auto Scaling group will launch instances into\. 

      If you don't specify any security groups in your launch template, Amazon EC2 uses the [default security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#DefaultSecurityGroup)\. By default, this security group doesn't allow inbound traffic from external networks\.

   1. \(Optional\) **Configure storage**: Update the storage configuration\. The default storage configuration is determined by the AMI and the instance type\. To configure advanced options, choose **Advanced**\.

      Use advanced options to review and configure the following: 

      1. **Volume type**: The type of volume depends on the instance type that you've chosen\. Each instance type has an associated root device volume, either an Amazon EBS volume or an instance store volume\. For more information, see [Amazon EC2 instance store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) and [Amazon EBS volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumes.html) in the *Amazon EC2 User Guide for Linux Instances*\.

      1. **Device name**: Specify a device name for the volume\.

      1. **Snapshot**: Enter the ID of the snapshot from which to create the volume\.

      1. **Size \(GiB\)**: For Amazon EBS\-backed volumes, specify a storage size\. If you're creating the volume from a snapshot and don't specify a volume size, the default is the snapshot size\.

      1. **Volume type**: For Amazon EBS volumes, choose the [volume type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)\.

      1. **IOPS**: With a Provisioned IOPS SSD volume, enter the maximum number of input/output operations per second \(IOPS\) that the volume should support\.

      1. **Delete on termination**: For Amazon EBS volumes, choose whether to delete the volume when the associated instance is terminated\. 

      1. **Encrypted**: Choose **Yes** to change the encryption state of an Amazon EBS volume\. The default effect of setting this parameter varies with the choice of volume source, as described in the following table\. In all cases, you must have permission to use the specified AWS KMS key\. For more information about specifying encrypted volumes, see [Amazon EBS encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Linux Instances*\.   
**Encryption outcomes**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html)

         \* If [encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default) is enabled, all newly created volumes \(whether or not the **Encrypted** parameter is set to **Yes**\) are encrypted using the default KMS key\. Setting both the **Encrypted** and **Key** parameters allows you to specify a non\-default KMS key\. 

      1. **Key**: If you chose **Yes** in the previous step, optionally enter the KMS key you want to use when encrypting the volumes\. Enter a KMS key that you previously created using the AWS Key Management Service\. You can paste the full ARN of any key that you have access to\. For information about setting up key policies for your customer managed keys, see the [AWS Key Management Service Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/) and the [Required AWS KMS key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md)\. 
**Note**  
Providing a KMS key without also setting the **Encrypted** parameter results in an error\. 

   1. For **Resource tags**, specify tags by providing key and value combinations\. You can tag the instances, the volumes, or both\. If you specify instance tags in your launch template and then you choose to propagate your Auto Scaling group's tags to its instances, all the tags are merged\. If the same tag key is specified for a tag in your launch template and a tag in your Auto Scaling group, then the tag value from the group takes precedence\. 

1. To change the default network interface, see [Change the default network interface](#change-network-interface)\. Skip this step if you want to keep the default network interface \(the primary network interface\)\.

1. To configure advanced settings, see [Configure advanced settings for your launch template](#advanced-settings-for-your-launch-template)\. Otherwise, choose **Create launch template**\. 

1. To create an Auto Scaling group, choose **Create Auto Scaling group** from the confirmation page\.

### Change the default network interface<a name="change-network-interface"></a>

This section shows you how to change the default network interface\. This allows you to define, for example, whether you want to assign a public IP address to each instance instead of defaulting to the auto\-assign public IP setting on the subnet\.

**Considerations and limitations**

When specifying a network interface, keep in mind the following considerations and limitations:
+ You must configure the security group as part of the network interface, and not in the **Security groups** section of the template\. You cannot specify security groups in both places\.
+ You cannot assign additional private IP addresses, known as secondary private IP addresses, to a network interface\. When an instance launches, a single private address is allocated to each network interface from the CIDR range of the subnet in which the instance is launched\. For more information on specifying CIDR ranges for your VPC or subnet, see the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\.
+ You can launch only one instance if you specify an existing network interface ID\. For this to work, you must use the AWS CLI or an SDK to create the Auto Scaling group\. When you create the group, you must specify the Availability Zone, but not the subnet ID\. Also, you can specify an existing network interface only if it has a device index of 0\. 
+ You cannot auto\-assign a public IP address if you specify more than one network interface\. You also cannot specify duplicate device indexes across network interfaces\. Note that both the primary and secondary network interfaces will reside in the same subnet\.

**To change the default network interface**

1. Under **Network settings**, expand **Advanced network configuration** and then choose **Add network interface**\.

1. Specify the primary network interface, paying attention to the following fields:

   1. **Device index**: Specify the device index\. Enter `0` for the primary network interface \(eth0\)\. 

   1. **Network interface**: Leave blank to create a new network interface when an instance is launched, or enter the ID of an existing network interface\. If you specify an ID, this limits your Auto Scaling group to one instance\. 

   1. **Description**: Enter a descriptive name\.

   1. **Subnet**: While you can choose to specify a subnet, it is ignored for Amazon EC2 Auto Scaling in favor of the settings of the Auto Scaling group\. 

   1. **Auto\-assign public IP**: Choose whether to automatically assign a [public IPv4 address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#public-ip-addresses) to the network interface with the device index of 0\. This setting takes precedence over settings that you configure for the subnets\. If you do not set a value, the default is to use the auto\-assign public IPv4 settings of the subnets that your instances are launched into\. 

   1. **Security groups**: Choose one or more [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)\. Each security group must be configured for the VPC that your Auto Scaling group will launch instances into\.

   1. **Delete on termination**: Choose whether the network interface is deleted when the Auto Scaling group scales in and terminates the instance to which the network interface is attached\. 

   1. **Elastic Fabric Adapter**: Indicates whether the network interface is an Elastic Fabric Adapter\. For more information, see [Elastic Fabric Adapter](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

   1. **Network card index**: Attaches the network interface to a specific network card when using an instance type that supports multiple network cards\. The primary network interface \(eth0\) must be assigned to network card index 0\. Defaults to 0 if not specified\. For more information, see [Network cards](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#network-cards) in the *Amazon EC2 User Guide for Linux Instances*\. 

1. To add a secondary network interface, choose **Add network interface**\.

### Configure advanced settings for your launch template<a name="advanced-settings-for-your-launch-template"></a>

You can define any additional capabilities that your Auto Scaling instances need\. For example, you can choose an IAM role that your application can use when it accesses other AWS resources or specify the instance user data that can be used to perform common automated configuration tasks after an instance starts\. 

The following steps discuss the most useful settings to pay attention to\. For more information about any of the settings under **Advanced details**, see [Creating a launch template](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#create-launch-template) in the *Amazon EC2 User Guide for Linux Instances*\.

**To configure advanced settings**

1. For **Advanced details**, expand the section to view the fields\.

1. For **Purchasing option**, you can choose **Request Spot Instances** to request Spot Instances at the Spot price, capped at the On\-Demand price, and choose **Customize** to change the default Spot Instance settings\. For an Auto Scaling group, you must specify a one\-time request with no end date \(the default\)\. For more information, see [Request Spot Instances for fault\-tolerant and flexible applications](launch-template-spot-instances.md)\. 
**Note**  
Amazon EC2 Auto Scaling lets you override the instance type in your launch template to create an Auto Scaling group that uses multiple instance types and launches Spot and On\-Demand Instances\. To do so, you must leave **Purchasing option** unspecified in your launch template\.  
If you try to create a mixed instances group using a launch template with **Purchasing option** specified, you get the following error\.  
Incompatible launch template: You cannot use a launch template that is set to request Spot Instances \(InstanceMarketOptions\) when you configure an Auto Scaling group with a mixed instances policy\. Add a different launch template to the group and try again\.  
For information about creating mixed instances groups, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.

1. For **IAM instance profile**, you can specify an AWS Identity and Access Management \(IAM\) instance profile to associate with the instances\. When you choose an instance profile, you associate the corresponding IAM role with the EC2 instances\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

1. For **Termination protection**, choose whether to protect instances from accidental termination\. When you enable termination protection, it provides additional termination protection, but it does not protect from Amazon EC2 Auto Scaling initiated termination\. To control whether an Auto Scaling group can terminate a particular instance, use [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

1. For **Detailed CloudWatch monitoring**, choose whether to enable the instances to publish metric data at 1\-minute intervals to Amazon CloudWatch\. Additional charges apply\. For more information, see [Configure monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\.

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

1. For **User data**, you can add shell scripts and cloud\-init directives to customize an instance at launch\. For more information, see [Run commands on your Linux instance at launch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) in the *Amazon EC2 User Guide for Linux Instances*\.
**Note**  
Running scripts at launch adds to the amount of time it takes for an instance to be ready for use\. However, you can allow extra time for the scripts to complete before the instance enters the `InService` state by adding a lifecycle hook to the Auto Scaling group\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

1. Choose **Create launch template**\.

1. To create an Auto Scaling group, choose **Create Auto Scaling group** from the confirmation page\.

## Create a launch template from an existing instance \(console\)<a name="create-launch-template-from-instance"></a>

**To create a launch template from an existing instance**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Instances**, choose **Instances**\.

1. Select the instance and choose **Actions**, **Image and templates**, **Create template from instance**\.

1. Provide a name and description\. 

1. Under **Auto Scaling guidance**, select the check box\. 

1. Adjust any settings as required, and choose **Create launch template**\. 

1. To create an Auto Scaling group, choose **Create Auto Scaling group** from the confirmation page\.

## Additional information<a name="create-launch-template-additional-info"></a>

For additional information about creating launch templates, see:
+ [Launching an instance from a launch template](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html) section of the *Amazon EC2 User Guide for Linux Instances*
+ [Auto scaling template snippets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-autoscaling.html) section of the *AWS CloudFormation User Guide*
+ [AWS::EC2::LaunchTemplate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-launchtemplate.html) section of the *AWS CloudFormation User Guide*

## Limitations<a name="create-launch-template-limitations"></a>
+ A launch template lets you configure a network type \(VPC or EC2\-Classic\), subnet, and Availability Zone\. However, these settings are ignored in favor of what is specified in the Auto Scaling group\. 
+ Because the subnet settings in your launch template are ignored in favor of what is specified in the Auto Scaling group, all of the network interfaces that are created for a given instance will be connected to the same subnet as the instance\. For other limitations on user\-defined network interfaces, see [Change the default network interface](#change-network-interface)\.
+ A launch template lets you configure additional settings in your Auto Scaling group to launch multiple instance types and combine On\-Demand and Spot purchase options, as described in [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\. Launching instances with such a combination is not supported:
  + If you specify a Spot Instance request in the launch template
  + In EC2\-Classic
+ Support for Dedicated Hosts \(host tenancy\) is only available if you specify a host resource group\. You cannot target a specific host ID or use host placement affinity\.