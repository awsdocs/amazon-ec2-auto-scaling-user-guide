# Create a launch template for an Auto Scaling group<a name="create-launch-template"></a>

Before you can create an Auto Scaling group using a launch template, you must create a launch template with the parameters required to launch an EC2 instance\. These parameters include the ID of the Amazon Machine Image \(AMI\) and an instance type\.

A launch template provides full functionality for Amazon EC2 Auto Scaling and also newer features of Amazon EC2 such as the current generation of Amazon EBS Provisioned IOPS volumes \(io2\), EBS volume tagging, T2 Unlimited instances, Elastic Inference, and Dedicated Hosts\.

To create new launch templates, use the following procedures\.

**Contents**
+ [Create your launch template \(console\)](#create-launch-template-for-auto-scaling)
  + [Change the default network interface settings](#change-network-interface)
  + [Modify the storage configuration](#modify-storage-configuration)
  + [Configure advanced settings for your launch template](#advanced-settings-for-your-launch-template)
+ [Create a launch template from an existing instance \(console\)](#create-launch-template-from-instance)
+ [Additional information](#create-launch-template-additional-info)
+ [Limitations](#create-launch-template-limitations)

**Important**  
Launch template parameters are not fully validated when you create the launch template\. If you specify incorrect values for parameters, or if you do not use supported parameter combinations, no instances can launch using this launch template\. Be sure to specify the correct values for the parameters and use supported parameter combinations\. For example, to launch instances with an Arm\-based AWS Graviton or Graviton2 AMI, you must specify an Arm\-compatible instance type\. 

## Create your launch template \(console\)<a name="create-launch-template-for-auto-scaling"></a>

The following steps describe how to configure your launch template: 
+ Specify the Amazon Machine Image \(AMI\) from which to launch the instances\.
+ Choose an instance type that is compatible with the AMI that you specify\. 
+ Specify the key pair to use when connecting to instances, for example, using SSH\.
+ Add one or more security groups to allow network access to the instances\.
+ Specify whether to attach additional volumes to each instance\. 
+ Add custom tags \(key\-value pairs\) to the instances and volumes\.

**To create a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Instances**, choose **Launch Templates**\.

1. Choose **Create launch template**\. Enter a name and provide a description for the initial version of the launch template\.

1. Under **Auto Scaling guidance**, select the check box to have Amazon EC2 provide guidance to help create a template to use with Amazon EC2 Auto Scaling\. 

1. Under **Launch template contents**, fill out each required field and any optional fields as needed\.

   1. **Application and OS Images \(Amazon Machine Image\)**: \(Required\) Choose the ID of the AMI for your instances\. You can search through all available AMIs, or select an AMI from the **Recents** or **Quick Start** list\. If you don't see the AMI that you need, choose **Browse more AMIs** to browse the full AMI catalog\.

      To choose a custom AMI, you must first create your AMI from a customized instance\. For more information, see [Create an AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-ami.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. For **Instance type**, choose a single instance type that's compatible with the AMI that you specified\.

      Alternatively, to launch an Auto Scaling group with multiple instance types, choose **Advanced**, **Specify instance type attributes**, and then specify the following options:
      + **Number of vCPUs**: Enter the minimum and maximum number of vCPUs\. To indicate no limits, enter a minimum of 0, and keep the maximum blank\. 
      + **Amount of memory \(MiB\)**: Enter the minimum and maximum amount of memory, in MiB\. To indicate no limits, enter a minimum of 0, and keep the maximum blank\. 
      + Expand **Optional instance type attributes** and choose **Add attribute** to further limit the types of instances that can be used to fulfill your desired capacity\. For information about each attribute, see [InstanceRequirementsRequest](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_InstanceRequirementsRequest.html) in the *Amazon EC2 API Reference*\. 
      + **Resulting instance types**: You can view the instance types that match the specified compute requirements, such as vCPUs, memory, and storage\.
      + To exclude instance types, choose **Add attribute**\. From the **Attribute** list, choose **Excluded instance types**\. From the **Attribute value** list, select the instance types to exclude\.

      For more information, see [Create an Auto Scaling group using attribute\-based instance type selection](create-asg-instance-type-requirements.md)\.

   1. **Key pair \(login\)**: For **Key pair name**, choose an existing key pair, or choose **Create new key pair** to create a new one\. For more information, see [Amazon EC2 key pairs and Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **Network settings**: For **Firewall \(security groups\)**, use one or more security groups, or keep this blank and configure one or more security groups as part of the network interface\. For more information, see [Amazon EC2 security groups for Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) in the *Amazon EC2 User Guide for Linux Instances*\.

      If you don't specify any security groups in your launch template, Amazon EC2 uses the default security group for the VPC that your Auto Scaling group will launch instances into\. By default, this security group doesn't allow inbound traffic from external networks\. For more information, see [Default security groups for your VPCs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#DefaultSecurityGroup) in the *Amazon VPC User Guide*\.

   1. Do one of the following:
      + Change the default network interface settings\. For example, you can enable or disable the public IPv4 addressing feature, which overrides the auto\-assign public IPv4 addresses setting on the subnet\. For more information, see [Change the default network interface settings](#change-network-interface)\.
      + Skip this step to keep the default network interface settings\. 

   1. Do one of the following:
      + Modify the storage configuration\. For more information, see [Modify the storage configuration](#modify-storage-configuration)\.
      + Skip this step to keep the default storage configuration\.

   1. For **Resource tags**, specify tags by providing key and value combinations\. If you specify instance tags in your launch template and then you choose to propagate your Auto Scaling group's tags to its instances, all the tags are merged\. If the same tag key is specified for a tag in your launch template and a tag in your Auto Scaling group, then the tag value from the group takes precedence\. 

1. \(Optional\) Configure advanced settings\. For more information, see [Configure advanced settings for your launch template](#advanced-settings-for-your-launch-template)\.

1. When you are ready to create the launch template, choose **Create launch template**\.

1. To create an Auto Scaling group, choose **Create Auto Scaling group** from the confirmation page\.

### Change the default network interface settings<a name="change-network-interface"></a>

This section shows you how to change the default network interface settings\. For example, you can define whether you want to assign a public IPv4 address to each instance instead of defaulting to the auto\-assign public IPv4 addresses setting on the subnet\.

**Considerations and limitations**

When changing the default network interface settings, keep in mind the following considerations and limitations:
+ You must configure the security groups as part of the network interface, not in the **Security groups** section of the template\. You cannot specify security groups in both places\.
+ You cannot assign secondary private IP addresses, known as *secondary IP addresses*, to a network interface\.
+ If you specify an existing network interface ID, you can launch only one instance\. To do this, you must use the AWS CLI or an SDK to create the Auto Scaling group\. When you create the group, you must specify the Availability Zone, but not the subnet ID\. Also, you can specify an existing network interface only if it has a device index of 0\. 
+ You cannot auto\-assign a public IPv4 address if you specify more than one network interface\. You also cannot specify duplicate device indexes across network interfaces\. Both the primary and secondary network interfaces reside in the same subnet\. For more information, see [Provide network connectivity for your Auto Scaling instances using Amazon VPC](asg-in-vpc.md)\. 
+ When an instance launches, a private address is automatically allocated to each network interface\. The address comes from the CIDR range of the subnet in which the instance is launched\. For information on specifying CIDR blocks \(or IP address ranges\) for your VPC or subnet, see the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\.

**To change the default network interface settings**

1. Under **Network settings**, expand **Advanced network configuration**\.

1. Choose **Add network interface** to configure the primary network interface, paying attention to the following fields:

   1. **Device index**: Keep the default value, 0, to apply your changes to the primary network interface \(eth0\)\.

   1. **Network interface**: Keep the default value, **New interface**, to have Amazon EC2 Auto Scaling automatically create a new network interface when an instance is launched\. Alternatively, you can choose an existing, available network interface with a device index of 0, but this limits your Auto Scaling group to one instance\. 

   1. **Description**: \(Optional\) Enter a descriptive name\.

   1. **Subnet**: Keep the default **Don't include in launch template** setting\. 

      If the AMI specifies a subnet for the network interface, this results in an error\. We recommend turning off **Auto Scaling guidance** as a workaround\. After you make this change, you will not receive an error message\. However, regardless of where the subnet is specified, the subnet settings of the Auto Scaling group take precedence and cannot be overridden\.

   1. **Auto\-assign public IP**: Change whether your network interface with a device index of 0 receives a public IPv4 address\. By default, instances in a default subnet receive a public IPv4 address, while instances in a nondefault subnet do not\. Select **Enable** or **Disable** to override the subnet's default setting\.

   1. **Security groups**: Choose one or more security groups for the network interface\. Each security group must be configured for the VPC that your Auto Scaling group will launch instances into\. For more information, see [Amazon EC2 security groups for Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **Delete on termination**: Choose **Yes** to delete the network interface when the instance is terminated, or choose **No** to keep the network interface\.

   1. **Elastic Fabric Adapter**: To support high performance computing \(HPC\) use cases, change the network interface into an Elastic Fabric Adapter network interface\. For more information, see [Elastic Fabric Adapter](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

   1. **Network card index**: Choose **0** to attach the primary network interface to the network card with a device index of 0\. If this option isn't available, keep the default value, **Don't include in launch template**\. Attaching the network interface to a specific network card is available only for supported instance types\. For more information, see [Network cards](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#network-cards) in the *Amazon EC2 User Guide for Linux Instances*\. 

1. To add a secondary network interface, choose **Add network interface**\.

### Modify the storage configuration<a name="modify-storage-configuration"></a>

You can modify the storage configuration for instances launched from an Amazon EBS\-backed AMI or an instance store\-backed AMI\. You can also specify additional EBS volumes to attach to the instances\. The AMI includes one or more volumes of storage, including the root volume \(**Volume 1 \(AMI Root\)**\)\. 

**To modify the storage configuration**

1. In **Configure storage**, modify the size or type of volume\. 

   If the value you specify for volume size is outside the limits of the volume type, or smaller than the snapshot size, an error message is displayed\. To help you address the issue, this message gives the minimum or maximum value that the field can accept\.

   Only volumes associated with an Amazon EBS\-backed AMI appear\. To display information about the storage configuration for an instance launched from an instance store\-backed AMI, choose **Show details** from the **Instance store volumes** section\.

   To specify all EBS volume parameters, switch to the **Advanced** view in the top right corner\. 

1. For advanced options, expand the volume that you want to modify and configure the volume as follows:

   1. **Storage type**: The type of volume \(EBS or ephemeral\) to associate with your instance\. The instance store \(ephemeral\) volume type is only available if you select an instance type that supports it\. For more information, see [Amazon EC2 instance store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) and [Amazon EBS volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumes.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **Device name**: Select from the list of available device names for the volume\.

   1. **Snapshot**: Select the snapshot from which to create the volume\. You can search for available shared and public snapshots by entering text into the **Snapshot** field\.

   1. **Size \(GiB\)**: For EBS volumes, you can specify a storage size\. If you have selected an AMI and instance that are eligible for the free tier, keep in mind that to stay within the free tier, you must stay under 30 GiB of total storage\. For more information, see [Constraints on the size and configuration of an EBS volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/volume_constraints.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **Volume type**: For EBS volumes, choose the volume type\. For more information, see [Amazon EBS volume types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **IOPS**: If you have selected a Provisioned IOPS SSD \(`io1` and `io2`\) or General Purpose SSD \(`gp3`\) volume type, then you can enter the number of I/O operations per second \(IOPS\) that the volume can support\. This is required for io1, io2, and gp3 volumes\. It is not supported for gp2, st1, sc1, or standard volumes\.

   1. **Delete on termination**: For EBS volumes, choose **Yes** to delete the volume when the instance is terminated, or choose **No** to keep the volume\.

   1. **Encrypted**: If the instance type supports EBS encryption, you can choose **Yes** to enable encryption for the volume\. If you have enabled encryption by default in this Region, encryption is enabled for you\. For more information, see [Amazon EBS encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) and [Encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default) in the *Amazon EC2 User Guide for Linux Instances*\. 

      The default effect of setting this parameter varies with the choice of volume source, as described in the following table\. In all cases, you must have permission to use the specified AWS KMS key\.   
**Encryption outcomes**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html)

      \* If encryption by default is enabled, all newly created volumes \(whether or not the **Encrypted** parameter is set to **Yes**\) are encrypted using the default KMS key\. If you set both the **Encrypted** and **KMS key** parameters, then you can specify a non\-default KMS key\. 

   1. **KMS key**: If you chose **Yes** for **Encrypted**, then you must select a customer managed key to use to encrypt the volume\. If you have enabled encryption by default in this Region, the default customer managed key is selected for you\. You can select a different key or specify the ARN of any customer managed key that you previously created using the AWS Key Management Service\.

1. To specify additional volumes to attach to the instances launched by this launch template, choose **Add new volume**\.

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

1. For **Elastic inference**, choose an elastic inference accelerator to attach to your EC2 CPU instance\. Additional charges apply\. For more information, see [Working with Amazon Elastic Inference](https://docs.aws.amazon.com/elastic-inference/latest/developerguide/working-with-ei.html) in the *Amazon Elastic Inference Developer Guide*\.

1. For **T2/T3 Unlimited**, choose whether to enable applications to burst beyond the baseline for as long as needed\. This field is only valid for T2, T3, and T3a instances\. Additional charges may apply\. For more information, see [Using an Auto Scaling group to launch a burstable performance instance as Unlimited](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-how-to.html#burstable-performance-instances-auto-scaling-grp) in the *Amazon EC2 User Guide for Linux Instances*\.

1. For **Placement group name**, you can specify a placement group in which to launch the instances\. Not all instance types can be launched in a placement group\. If you configure an Auto Scaling group using a CLI command that specifies a different placement group, the placement group for the Auto Scaling group takes precedence\.

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

For the procedures for creating an Auto Scaling group with a launch template, see the following topics: 
+ [Create an Auto Scaling group using a launch template](create-asg-launch-template.md)
+ [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)
+ [Create an Auto Scaling group using attribute\-based instance type selection](create-asg-instance-type-requirements.md)

## Limitations<a name="create-launch-template-limitations"></a>
+ Amazon EC2 lets you configure a subnet in a launch template\. However, the subnet settings of the Auto Scaling group take precedence over the subnet settings of the launch template\.
+ Because the subnet settings in your launch template are ignored in favor of what is specified in the Auto Scaling group, all of the network interfaces that are created for a given instance will be connected to the same subnet as the instance\. For additional limitations on user\-defined network interfaces, see [Change the default network interface settings](#change-network-interface)\.
+ A launch template lets you configure additional settings in your Auto Scaling group to launch multiple instance types and combine On\-Demand and Spot purchase options, as described in [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\. Launching instances with such a combination is not supported if you specify a Spot Instance request in the launch template\.
+ Support for Dedicated Hosts \(host tenancy\) is only available if you specify a host resource group\. You cannot target a specific host ID or use host placement affinity\.