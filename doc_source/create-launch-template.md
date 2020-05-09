# Creating a Launch Template for an Auto Scaling Group<a name="create-launch-template"></a>

Before you can create an Auto Scaling group using a launch template, you must create a launch template that includes the parameters required to launch an EC2 instance, such as the ID of the Amazon Machine Image \(AMI\) and an instance type\.

The following procedure works for creating a new launch template\. The new template uses parameters that you define \(starting from scratch\) or an existing launch template\. After you create your launch template, you can create the Auto Scaling group by following the instructions in [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

**Prerequisites**

For information about the required IAM permissions, see [Controlling the Use of Launch Templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#launch-template-permissions) in the *Amazon EC2 User Guide for Linux Instances* and the [Launch Template Support](ec2-auto-scaling-launch-template-permissions.md) topic in this guide\.

**Considerations**

Keep the following considerations in mind when creating a launch template for use with an Auto Scaling group:
+ Launch templates give you the flexibility of launching one type of instance, or a combination of instance types and On\-Demand and Spot purchase options\. For more information, see [Auto Scaling Groups with Multiple Instance Types and Purchase Options](asg-purchase-options.md)\. Launching instances with such a combination is not supported:
  + If you specify a Spot Instance request in **Additional Details**
  + In EC2\-Classic
+ If you configure a network type \(VPC or EC2\-Classic\), subnet, and Availability Zone for your template, these settings are ignored in favor of what is specified in the Auto Scaling group\. 
+ If you specify a network interface, you must configure the security group as part of the network interface, and not in the **Security Groups** section of the template\. 
+ You cannot specify multiple network interfaces\.
+ You cannot assign specific private IP addresses\. When an instance launches, a private address is allocated from the CIDR range of the subnet in which the instance is launched\. For more information on specifying CIDR ranges for your VPC or subnet, see the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\.
+ To specify an existing network interface to use, its device index must be 0 \(eth0\)\. For this scenario, you must use the CLI or API to create the Auto Scaling group\. When you create the group using the CLI create\-auto\-scaling\-group command or API CreateAutoScalingGroup action, you must specify the Availability Zones parameter instead of the subnet \(VPC zone identifier\) parameter\. 
+ You cannot use host placement affinity or target a specific host by choosing a host ID\.
+ Support for host tenancy \(Dedicated Hosts\) is only available if you specify a host resource group\. For more information, see [Host Resource Groups](https://docs.aws.amazon.com/license-manager/latest/userguide/host-resource-groups.html) in the *AWS License Manager User Guide*\. Note that each AMI based on a license configuration association can be mapped to only one host resource group at a time\. 

**To create a new launch template for an Auto Scaling group \(new EC2 console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Launch Templates**\.

1. Choose **Create launch template**\. Enter a name and provide a description for the initial version of the launch template\.

1. Under **Launch template contents**, fill out each required field and any optional fields to use as your instance launch specification\.

   1. \(Required\) **Amazon machine image \(AMI\)**: Choose the ID of the AMI from which to launch the instances\. You can search through all available AMIs, or from the **Quick Start** list, select from one of the commonly used AMIs in the list\. If you don't see the AMI that you need, you can [find a suitable AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html), make note of its ID, and specify it as a custom value\.

   1. \(Recommended\) **Instance type**: Choose the [instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)\. Ensure that the instance type is compatible with the AMI you've specified\. 

   1. \(Recommended\) **Key pair \(login\)**: Specify the [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) to use when connecting to your instances\.

1. Under **Network settings**, to configure one or more VPC or EC2\-Classic security groups, complete the following steps:

   1. **Networking platform**: Choose whether to launch instances into a VPC or EC2\-Classic, if applicable\. However, the network type and Availability Zone settings of the launch template are ignored for Amazon EC2 Auto Scaling in favor of the settings of the Auto Scaling group\. 

   1. **Security groups**: Choose one or more [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html), or leave blank to configure one or more VPC security groups as part of the network interface\. You cannot specify security groups in both places\. If you're using EC2\-Classic, you must use security groups created specifically for EC2\-Classic\.

1. Each instance that is launched has an associated root device volume, either an Amazon EBS volume or an instance store volume\. You can specify additional EBS volumes or instance store volumes to attach to an instance when it is launched\. If you choose to specify volumes to attach to the instances not including the volumes specified by the AMI, under **Storage \(Volumes\)**, choose **Add new volume** and provide the following information:

   1. **Volume type**: The type of volume depends on the instance type that you've chosen\. For more information, see [Amazon EC2 Instance Store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) and [Amazon EBS Volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumes.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **Device name**: Specify a device name for the volume\.

   1. **Snapshot**: Enter the ID of the snapshot from which to create the volume\.

   1. **Size \(GiB\)**: For Amazon EBS\-backed volumes, specify a storage size\. If you're creating the volume from a snapshot and don't specify a volume size, the default is the snapshot size\.

   1. **Volume type**: For Amazon EBS volumes, choose the [volume type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)\.

   1. **IOPS**: With a Provisioned IOPS SSD volume, enter the maximum number of input/output operations per second \(IOPS\) that the volume should support\.

   1. **Delete on termination**: For Amazon EBS volumes, choose whether to delete the volume when the associated instance is terminated\. 

   1. **Encrypted**: Choose **Yes** to change the encryption state of an Amazon EBS volume\. The default effect of setting this parameter varies with the choice of volume source, as described in the table below\. You must in all cases have permission to use the specified CMK\. For more information about specifying encrypted volumes, see [Amazon EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Linux Instances*\.   
**Encryption Outcomes**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html)

      \* If [encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default) is enabled, all newly created volumes \(whether or not the **Encrypted** parameter is set to **Yes**\) are encrypted using the default CMK\. Setting both the **Encrypted** and **Key** parameters allows you to specify a non\-default CMK\. 

   1. \(Optional\) **Key**: If you chose **Yes** in the previous step, enter the customer master key \(CMK\) you want to use when encrypting the volumes\. Enter any CMK that you previously created using the AWS Key Management Service\. You can paste the full ARN of any key that you have access to\. For more information, see the [AWS Key Management Service Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/) and the [Required CMK Key Policy for Use with Encrypted Volumes](key-policy-requirements-EBS-encryption.md) topic in this guide\. Note: Amazon EBS does not support asymmetric CMKs\. 
**Note**  
Providing a CMK without also setting the **Encrypted** parameter results in an error\. 

1. For **Instance tags**, specify tags by providing key and value combinations\. You can tag the instances, the volumes, or both\.

1. Under **Network interfaces**, choose **Add network interface** and provide the following optional information\. When launching instances in a VPC, you can specify the default network interface, called the *primary network interface* \(eth0\)\. You can only specify one network interface\. Pay attention to the following fields:

   1. **Device**: Specify `eth0` as the device name \(the device for which the device index is 0\)\. 

   1. **Network interface**: Leave blank to let AWS create a new network interface when an instance is launched, or enter the ID of an existing network interface\. If you specify an ID, this limits your Auto Scaling group to one instance\. 

   1. **Description**: Enter a descriptive name\.

   1. **Subnet**: While you may choose to specify a subnet, it is ignored for Amazon EC2 Auto Scaling in favor of the settings of the Auto Scaling group\. 

   1. **Auto\-assign public IP**: Specify whether to automatically assign a public IP address to the network interface with the device index of `eth0`\. This setting can only be enabled for a single, new network interface\. For more information, see [Assigning a Public IPv4 Address During Instance Launch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#public-ip-addresses) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. **Security group ID**: Enter the IDs of one or more [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html) with which to associate the primary network interface \(eth0\)\. Each security group must be configured for the VPC that your Auto Scaling group will launch instances into\. Separate the entries with commas\.

   1. **Delete on termination**: Choose whether the network interface is deleted when the Auto Scaling group scales in and terminates the instance to which the network interface is attached\.

1. For **Advanced Details**, expand the section to view the fields and specify any additional parameters for the instances\. 
   + **Purchasing option**: You have the option to request Spot Instances and specify the maximum price you are willing to pay per instance hour\. For this to work with an Auto Scaling group, you must specify a one\-time request with no end date\. For more information, see [Launching Spot Instances in Your Auto Scaling Group](asg-launch-spot-instances.md)\.
**Important**  
If you plan to specify multiple instance types and purchase options when you configure your Auto Scaling group, leave these fields empty\. For more information, see [Auto Scaling Groups with Multiple Instance Types and Purchase Options](asg-purchase-options.md)\.
   + **IAM instance profile**: Specify an AWS Identity and Access Management \(IAM\) instance profile to associate with the instances\. For more information, see [IAM Role for Applications That Run on Amazon EC2 Instances](us-iam-role.md)\.
   + **Shutdown behavior**: You can leave this field blank because it is ignored for Amazon EC2 Auto Scaling\. The default behavior of Amazon EC2 Auto Scaling is to terminate the instance\. 
   + **Stop \- Hibernate behavior**: You can leave this field blank because it is ignored for Amazon EC2 Auto Scaling\. The default behavior of Amazon EC2 Auto Scaling is to terminate the instance\. 
   + **Termination protection**: Provides additional termination protection but is ignored for Amazon EC2 Auto Scaling when scaling in your Auto Scaling group\. To control whether an Auto Scaling group can terminate a particular instance when scaling in, use [Instance Scale\-In Protection](as-instance-termination.md#instance-protection)\.
   + **Detailed CloudWatch monitoring**: Choose whether to enable detailed monitoring of the instances using Amazon CloudWatch\. Additional charges apply\. For more information, see [Monitoring Your Auto Scaling Groups and Instances Using Amazon CloudWatch](as-instance-monitoring.md)\.
   + **T2/T3 Unlimited**: \(Only valid for T2 and T3 instances\) Choose whether to enable applications to burst beyond the baseline for as long as needed\. Additional charges may apply\. For more information, see [Using an Auto Scaling Group to Launch a Burstable Performance Instance as Unlimited](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-how-to.html#burstable-performance-instances-auto-scaling-grp) in the *Amazon EC2 User Guide for Linux Instances*\.
   + **Placement group name**: Specify a placement group in which to launch the instances\. Not all instance types can be launched in a placement group\. If you configure an Auto Scaling group using a CLI command that specifies a different placement group, the setting is ignored in favor of the one specified for the Auto Scaling group\.
   + **EBS\-optimized instance**: Provides additional, dedicated capacity for Amazon EBS I/O\. Not all instance types support this feature, and additional charges apply\. 
   + **Tenancy**: You can choose to run your instances on shared hardware \(**Shared**\), on dedicated hardware \(**Dedicated**\), or, when using a host resource group, on Dedicated Hosts \(**Dedicated host**\)\. Additional charges may apply\. 
   + **Tenancy host resource group**: Specify a host resource group for a BYOL AMI to use on Dedicated Hosts\. For more information, see [Host Resource Groups](https://docs.aws.amazon.com/license-manager/latest/userguide/host-resource-groups.html) in the *AWS License Manager User Guide*\. Note: You do not have to have already allocated Dedicated Hosts in your account before you use this feature\. Your instances will automatically launch onto Dedicated Hosts regardless\.
   + **RAM disk ID**: The ID of the RAM disk associated with the AMI\. Only valid for paravirtual \(PV\) AMIs\. 
   + **Kernel ID**: The ID of the kernel associated with the AMI\. Only valid for paravirtual \(PV\) AMIs\. 
   + **License configurations**: You can specify the license configuration to use\. 
   + **User data**: You can specify user data to configure an instance during launch, or to run a configuration script\. 

1. Choose **Create launch template**\.

If you are not currently using the new EC2 console, you can create a launch template using the old EC2 console instead\. 

**To create a new launch template for an Auto Scaling group \(old EC2 console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Launch Templates**\.

1. Choose **Create a new template**\. Enter a name and provide a description for the initial version of the launch template\. 

1. If you choose to create the new template based on another template:

   1. For **Source template**, choose an existing launch template\.

   1. Choose the launch template version on which to base the new launch template from **Source template version**\. 

1. Under **Launch template contents**, provide the following information\.

   1. **AMI ID**: Choose the ID of the Amazon Machine Image \(AMI\) from which to launch the instances\. You can search through all available AMIs using the **Search for AMI** dialog\. From the **Quick Start** list, select from one of the commonly used AMIs in the list\. If you don't see the AMI that you need, select the **AWS Marketplace** or **Community AMIs** list to [find a suitable AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html)\.

   1. **Instance type**: Choose the [instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)\. Ensure that the instance type is compatible with the AMI you've specified\. 

   1. **Key pair name**: Specify the [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) to use when connecting to your instances\.

   1. **Network type**: You can choose to specify whether to launch instances into a VPC or EC2\-Classic, if applicable\. However, the network type and Availability Zone settings of the launch template are ignored for Amazon EC2 Auto Scaling in favor of the settings of the Auto Scaling group\. 

   1. **Security Groups**: Choose one or more VPC or EC2\-Classic [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html), or leave blank to configure one or more VPC security groups as part of the network interface\. You cannot specify security groups in both places\. If you're using EC2\-Classic, you must use security groups created specifically for EC2\-Classic\.

1. Under **Network interfaces**, specify the primary network interface \(eth0\)\. You can only specify one network interface\. 

1. Under **Storage \(Volumes\)**, specify additional storage volumes for your instance, using a block device mapping\. 

1. For **Instance tags**, specify tags by providing key and value combinations\. You can tag the instances, the volumes, or both\.

1. For **Advanced Details**, expand the section to view the fields and specify any additional parameters for the instances\.

1. Choose **Create launch template**\.

**To create a launch template from an existing instance**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Select the instance and choose **Actions**, **Create Template from Instance**\.

1. Provide a name and description\. Adjust any other launch parameters as required, and choose **Create Launch Template**\.

**To create a launch template using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2LaunchTemplate.html](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2LaunchTemplate.html) \(AWS Tools for Windows PowerShell\)

Create a launch template using the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command as follows\. Specify a value for `Groups` that corresponds to security groups for the VPC that your Auto Scaling group will launch instances into\. Specify the VPC and subnets as properties of the Auto Scaling group\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
--launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"AssociatePublicIpAddress":true,"Groups":["sg-7c227019"],"DeleteOnTermination":true}],"ImageId":"ami-01e24be29428c15b2","InstanceType":"t2.micro","TagSpecifications": [{"ResourceType":"instance","Tags":[{"Key":"purpose","Value":"webserver"}]}]}'
```

Use the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html) command to describe the launch template *my\-template\-for\-auto\-scaling*\.

```
aws ec2 describe-launch-templates --launch-template-names my-template-for-auto-scaling
```

Use the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html) command to describe the versions of the specified launch template *my\-template\-for\-auto\-scaling*\.

```
aws ec2 describe-launch-template-versions --launch-template-id lt-068f72b72934aff71
```

The following is an example response:

```
{
    "LaunchTemplateVersions": [
        {
            "VersionDescription": "version1",
            "LaunchTemplateId": "lt-068f72b72934aff71",
            "LaunchTemplateName": "my-template-for-auto-scaling",
            "VersionNumber": 1,
            "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
            "LaunchTemplateData": {
                "TagSpecifications": [
                    {
                        "ResourceType": "instance",
                        "Tags": [
                            {
                                "Key": "purpose",
                                "Value": "webserver"
                            }
                        ]
                    }
                ],
                "ImageId": "ami-01e24be29428c15b2",
                "InstanceType": "t2.micro",
                "NetworkInterfaces": [
                    {
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "Groups": [
                            "sg-7c227019"
                        ],
                        "AssociatePublicIpAddress": true
                    }
                ]
            },
            "DefaultVersion": true,
            "CreateTime": "2019-02-28T19:52:27.000Z"
        }
    ]
}
```