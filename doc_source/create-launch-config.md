# Creating a launch configuration<a name="create-launch-config"></a>

**Important**  
We recommend that you create Auto Scaling groups from launch templates to ensure that you're getting the latest features from Amazon EC2\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.

When you create a launch configuration, you must specify information about the EC2 instances to launch\. Include the Amazon Machine Image \(AMI\), instance type, key pair, security groups, and block device mapping\. Alternatively, you can create a launch configuration using attributes from a running EC2 instance\. For more information, see [Creating a launch configuration using an EC2 instance](create-lc-with-instanceID.md)\.

After you create a launch configuration, you can create an Auto Scaling group\. For more information, see [Creating an Auto Scaling group using a launch configuration](create-asg.md)\.

An Auto Scaling group is associated with one launch configuration at a time, and you can't modify a launch configuration after you've created it\. Therefore, if you want to change the launch configuration for an existing Auto Scaling group, you must update it with the new launch configuration\. For more information, see [Changing the launch configuration for an Auto Scaling group](change-launch-config.md)\.

**Topics**
+ [Creating your launch configuration \(console\)](#create-launch-configuration)
+ [Creating a launch configuration \(AWS CLI\)](#create-launch-configuration-aws-cli)
+ [Configuring the instance metadata options](#launch-configurations-imds)

## Creating your launch configuration \(console\)<a name="create-launch-configuration"></a>

**To create a launch configuration \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\. 

1.  In the navigation bar, select your AWS Region\. 

1. Choose **Create launch configuration**, and enter a name for your launch configuration\. 

1. For **Amazon machine image \(AMI\) **, choose an AMI\. To find a specific AMI, you can [find a suitable AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html), make note of its ID, and enter the ID as search criteria\.

1. For **Instance type**, select a hardware configuration for your instances\.

1. In **Additional configuration**, pay attention to the following fields:

   1. \(Optional\) For **Purchasing option**, you can choose **Request Spot Instances** to request Spot Instances at the Spot price, capped at the On\-Demand price\. Optionally, you can specify a maximum price per instance hour for your Spot Instances\. For more information, see [Launching Spot Instances in your Auto Scaling group](asg-launch-spot-instances.md)\. 

   1. \(Optional\) For **IAM instance profile**, choose a role to associate with the instances\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

   1. \(Optional\) For **Monitoring**, choose whether to enable the instances to publish metric data at 1\-minute intervals to Amazon CloudWatch by enabling detailed monitoring\. Additional charges apply\. For more information, see [Configuring monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\.

   1. \(Optional\) For **User data**, you can specify user data to configure an instance during launch, or to run a configuration script after the instance starts\. 

   1. \(Optional\) For **Advanced Details**, **IP Address Type**, choose whether to assign a [public IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#public-ip-addresses) to the group's instances\. This setting takes precedence over settings you configure for the subnets\. If you do not set a value, the default is to use the auto\-assign public IP settings of the subnets that your instances are launched into\.

1. \(Optional\) For **Storage \(volumes\)**, if you don't need additional storage, you can skip this section\. Otherwise, to specify volumes to attach to the instances in addition to the volumes specified by the AMI, choose **Add new volume**\. Then choose the desired options and associated values for **Devices**, **Snapshot**, **Size**, **Volume type**, **IOPS**, **Throughput**, **Delete on termination**, and **Encrypted**\.

1. For **Security groups**, create or select the security group to associate with the group's instances\. If you leave the **Create a new security group** option selected, a default SSH rule is configured for Amazon EC2 instances running Linux\. A default RDP rule is configured for Amazon EC2 instances running Windows\. 

1. For **Key pair \(login\)**, choose an option under **Key pair options**\. 

   If you've already configured an Amazon EC2 instance key pair, you can choose it here\. 

   If you don't already have an Amazon EC2 instance key pair, choose **Create a new key pair** and give it a recognizable name\. Choose **Download key pair** to download the key pair to your computer\. 
**Important**  
If you need to connect to your instances, do not choose **Proceed without a key pair**\.

1. Select the acknowledgment check box, and then choose **Create launch configuration**\.

**To create a launch configuration \(old console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select your AWS Region\. 

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\. 

1. On the next page, choose **Create launch configuration**\.

1. On the **Choose AMI** page, select an AMI\.

1. On the **Choose Instance Type** page, select a hardware configuration for your instance\. Choose **Next: Configure details**\.

1. On the **Configure Details** page, do the following:

   1. For **Name**, enter a name for your launch configuration\.

   1. \(Optional\) For **Purchasing option**, you can request Spot Instances at the Spot price, capped at the On\-Demand price\. Optionally, you can specify a maximum price per instance hour for your Spot Instances\. For more information, see [Launching Spot Instances in your Auto Scaling group](asg-launch-spot-instances.md)\.

   1. \(Optional\) For **IAM role**, select a role to associate with the instances\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

   1. \(Optional\) By default, basic monitoring is enabled for your Auto Scaling instances\. To enable detailed monitoring for your Auto Scaling instances, select **Enable CloudWatch detailed monitoring**\. For more information, see [Configuring monitoring for Auto Scaling instances](enable-as-instance-metrics.md)\.

   1. \(Optional\) For **User data**, you can specify user data to configure an instance during launch, or to run a configuration script after the instance starts\.

   1. \(Optional\) For **Advanced Details**, **IP Address Type**, choose whether to assign a [public IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#public-ip-addresses) to the group's instances\. This setting takes precedence over settings you configure for the subnets\. If you do not set a value, the default is to use the auto\-assign public IP settings of the subnets that your instances are launched into\. 

   1. Choose **Skip to review**\.

1. On the **Review** page, choose **Edit security groups**\. Follow the instructions to choose an existing security group, and then choose **Review**\.

1. On the **Review** page, choose **Create launch configuration**\.

1. For **Select an existing key pair or create a new key pair**, select one of the listed options\. Select the acknowledgment check box, and then choose **Create launch configuration**\.
**Warning**  
If you need to connect to your instances, do not choose **Proceed without a key pair**\.

## Creating a launch configuration \(AWS CLI\)<a name="create-launch-configuration-aws-cli"></a>

**To create a launch configuration using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASLaunchConfiguration.html](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASLaunchConfiguration.html) \(AWS Tools for Windows PowerShell\)

## Configuring the instance metadata options<a name="launch-configurations-imds"></a>

Amazon EC2 Auto Scaling supports configuring the Instance Metadata Service \(IMDS\) in launch configurations\. This gives you the option of using launch configurations to configure the Amazon EC2 instances in your Auto Scaling groups to require Instance Metadata Service Version 2 \(IMDSv2\), which is a session\-oriented method for requesting instance metadata\. For details about IMDSv2's advantages, see this article on the AWS Blog about [enhancements to add defense in depth to the EC2 instance metadata service](http://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/)\. 

You can configure IMDS to support both IMDSv2 and IMDSv1 \(the default\), or to require the use of IMDSv2\. If you are using the AWS CLI or an AWS SDK to configure IMDS, you must use the latest version of the AWS CLI or the SDK to require the use of IMDSv2\.

You can configure your launch configuration for the following: 
+ Require the use of IMDSv2 when requesting instance metadata
+ Specify the `PUT` response hop limit 
+ Turn off access to instance metadata

You can find more details on configuring the Instance Metadata Service in the following topic: [Configuring the instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) in the *Amazon EC2 User Guide for Linux Instances*\.

Use the following procedure to configure IMDS options in a launch configuration\. After you create your launch configuration, you can associate it with your Auto Scaling group\. If you associate the launch configuration with an existing Auto Scaling group, the existing launch configuration is disassociated from the Auto Scaling group, and existing instances will require replacement to use the IMDS options that you specified in the new launch configuration\. For more information, see [Changing the launch configuration for an Auto Scaling group](change-launch-config.md)\.

**To configure IMDS in a launch configuration \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\. 

1.  In the navigation bar, select your AWS Region\. 

1. Choose **Create launch configuration**, and create the launch configuration the usual way\. Include the ID of the Amazon Machine Image \(AMI\), the instance type, and optionally, a key pair, one or more security groups, and any additional EBS volumes or instance store volumes for your instances\.

1. To configure instance metadata options for all of the instances associated with this launch configuration, in **Additional configuration**, under **Advanced details**, do the following:

   1. For **Metadata accessible**, choose whether to enable or disable access to the HTTP endpoint of the instance metadata service\. By default, the HTTP endpoint is enabled\. If you choose to disable the endpoint, access to your instance metadata is turned off\. You can specify the condition to require IMDSv2 only when the HTTP endpoint is enabled\.

   1. For **Metadata version**, you can choose to require the use of Instance Metadata Service Version 2 \(IMDSv2\) when requesting instance metadata\. If you do not specify a value, the default is to support both IMDSv1 and IMDSv2\.

   1. For **Metadata token response hop limit**, you can set the allowable number of network hops for the metadata token\. If you do not specify a value, the default is 1\.

1. When you have finished, choose **Create launch configuration**\.

**To require the use of IMDSv2 in a launch configuration using the AWS CLI**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command with `--metadata-options` set to `HttpTokens=required`\. When you specify a value for `HttpTokens`, you must also set `HttpEndpoint` to enabled\. Because the secure token header is set to required for metadata retrieval requests, this opts in the instance to require using IMDSv2 when requesting instance metadata\. 

```
aws autoscaling create-launch-configuration \
  --launch-configuration-name my-lc-with-imdsv2 \
  --image-id ami-01e24be29428c15b2 \
  --instance-type t2.micro \
      ...    
  --metadata-options "HttpEndpoint=enabled,HttpTokens=required"
```

**To turn off access to instance metadata**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command to turn off access to instance metadata\. You can turn access back on later by using the [modify\-instance\-metadata\-options](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-metadata-options.html) command\. 

```
aws autoscaling create-launch-configuration \
  --launch-configuration-name my-lc-with-imds-disabled \
  --image-id ami-01e24be29428c15b2 \
  --instance-type t2.micro \
      ...    
  --metadata-options "HttpEndpoint=disabled"
```