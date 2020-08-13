# Launching Auto Scaling instances in a VPC<a name="asg-in-vpc"></a>

Amazon Virtual Private Cloud \(Amazon VPC\) enables you to define a virtual networking environment in a private, isolated section of the AWS Cloud\. You have complete control over your virtual networking environment\. For more information, see the *[Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)*\.

Within a virtual private cloud \(VPC\), you can launch AWS resources such as an Auto Scaling group\. An Auto Scaling group in a VPC works essentially the same way as it does on Amazon EC2 and supports the same set of features\.

A subnet in Amazon VPC is a subdivision within an Availability Zone defined by a segment of the IP address range of the VPC\. Using subnets, you can group your instances based on your security and operational needs\. A subnet resides entirely within the Availability Zone it was created in\. You launch Auto Scaling instances within the subnets\.

To enable communication between the internet and the instances in your subnets, you must create an internet gateway and attach it to your VPC\. An internet gateway enables your resources within the subnets to connect to the internet through the Amazon EC2 network edge\. If a subnet's traffic is routed to an internet gateway, the subnet is known as a *public* subnet\. If a subnet's traffic is not routed to an internet gateway, the subnet is known as a *private* subnet\. Use a public subnet for resources that must be connected to the internet, and a private subnet for resources that need not be connected to the internet\.

Before you can launch your Auto Scaling instances in a new VPC, you must first create your VPC environment\. After you create your VPC and subnets, you launch Auto Scaling instances within the subnets\. The easiest way to create a VPC with one public subnet is to use the VPC wizard\. For more information, see the [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/)\.

**Topics**
+ [Default VPC](#as-defaultVPC)
+ [IP addressing in a VPC](#as-vpc-ipaddress)
+ [Instance placement tenancy](#as-vpc-tenancy)
+ [Linking EC2\-Classic instances to a VPC](#as-ClassicLink)
+ [More resources for learning about VPCs](#auto-scaling-resources-about-vpcs)

## Default VPC<a name="as-defaultVPC"></a>

If you created your AWS account after December 4, 2013 or you are creating your Auto Scaling group in a new AWS Region, we create a default VPC for you\. Your default VPC comes with a default subnet in each Availability Zone\. If you have a default VPC, your Auto Scaling group is created in the default VPC by default\.

For information about default VPCs and checking whether your account comes with a default VPC, see [Your default VPC and subnets](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html#launching-into) in the *Amazon VPC Developer Guide*\. 

## IP addressing in a VPC<a name="as-vpc-ipaddress"></a>

When you launch your Auto Scaling instances in a VPC, your instances are automatically assigned a private IP address in the address range of the subnet\. This enables your instances to communicate with other instances in the VPC\.

You can configure your launch configuration to assign public IP addresses to your instances\. Assigning public IP addresses to your instances enables them to communicate with the internet or other services in AWS\.

When you enable public IP addresses for your instances and launch them into a subnet that is configured to automatically assign IPv6 addresses, they receive both IPv4 and IPv6 addresses\. Otherwise, they receive only IPv4 addresses\. For more information, see [IPv6 addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#ipv6-addressing) in the *Amazon EC2 User Guide for Linux Instances*\.

## Instance placement tenancy<a name="as-vpc-tenancy"></a>

Tenancy defines how EC2 instances are distributed across physical hardware and affects pricing\. There are three tenancy options available: 
+ Shared \(`default`\) — Multiple AWS accounts may share the same physical hardware\. 
+ Dedicated Instance \(`dedicated`\) — Your instance runs on single\-tenant hardware\. 
+ Dedicated Host \(`host`\) — Your instance runs on a physical server with EC2 instance capacity fully dedicated to your use, an isolated server with configurations that you can control\. 

**Important**  
You can configure tenancy for EC2 instances using a launch configuration or launch template\. However, the `host` tenancy value cannot be used with a launch configuration\. Use the `default` or `dedicated` tenancy values only\. To use a tenancy value of `host`, you must use a launch template\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.

Dedicated Instances are physically isolated at the host hardware level from instances that aren't dedicated and from instances that belong to other AWS accounts\. When you create a VPC, by default its tenancy attribute is set to `default`\. In such a VPC, you can launch instances with a tenancy value of `dedicated` so that they run as single\-tenancy instances\. Otherwise, they run as shared\-tenancy instances by default\. If you set the tenancy attribute of a VPC to `dedicated`, all instances launched in the VPC run as single\-tenancy instances\. For more information, see [Dedicated instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html) in the *Amazon EC2 User Guide for Linux Instances*\. For pricing information, see the [Amazon EC2 dedicated instances](https://aws.amazon.com/ec2/purchasing-options/dedicated-instances/) product page\.

When you create a launch configuration, the default value for the instance placement tenancy is `null` and the instance tenancy is controlled by the tenancy attribute of the VPC\. The following table summarizes the instance placement tenancy of the Auto Scaling instances launched in a VPC\.


| Launch configuration tenancy | VPC tenancy = default | VPC tenancy = dedicated | 
| --- | --- | --- | 
|  not specified  |  shared\-tenancy instance  |  dedicated instance  | 
|   `default`   |  shared\-tenancy instance  |  dedicated instance  | 
|   `dedicated`   |  dedicated instance  |  dedicated instance  | 

**To configure tenancy using a launch configuration \(AWS CLI\)**  
You can specify the instance placement tenancy for your launch configuration as `default` or `dedicated` using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command with the `--placement-tenancy` option\. For example, the following command sets the launch configuration tenancy to `dedicated`\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-launch-config --placement-tenancy dedicated --image-id ...
```

You can use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to verify the instance placement tenancy of the launch configuration\.

```
aws autoscaling describe-launch-configurations --launch-configuration-names my-launch-config
```

The following is example output for a launch configuration that creates dedicated instances\. The `PlacementTenancy` parameter is only part of the output for this command when you explicitly set the instance placement tenancy\.

```
{
    "LaunchConfigurations": [
        {
            "UserData": null,
            "EbsOptimized": false,
            "PlacementTenancy": "dedicated",
            "LaunchConfigurationARN": "arn",
            "InstanceMonitoring": {
                "Enabled": true
            },
            "ImageId": "ami-b5a7ea85",
            "CreatedTime": "2015-03-08T23:39:49.011Z",
            "BlockDeviceMappings": [],
            "KeyName": null,
            "SecurityGroups": [],
            "LaunchConfigurationName": "my-launch-config",
            "KernelId": null,
            "RamdiskId": null,
            "InstanceType": "m3.medium"
        }
    ]
```

## Linking EC2\-Classic instances to a VPC<a name="as-ClassicLink"></a>

If you are launching the instances in your Auto Scaling group in EC2\-Classic, you can link them to a VPC using *ClassicLink*\. ClassicLink enables you to associate one or more security groups for the VPC with the EC2\-Classic instances in your Auto Scaling group\. It enables communication between these linked EC2\-Classic instances and instances in the VPC using private IP addresses\. For more information, see [ClassicLink](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html) in the *Amazon EC2 User Guide for Linux Instances*\.

If you have running EC2\-Classic instances in your Auto Scaling group, you can link them to a VPC with ClassicLink enabled\. For more information, see [Linking an instance to a VPC](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html#classiclink-link-instance) in the *Amazon EC2 User Guide for Linux Instances*\. Alternatively, you can update the Auto Scaling group to use a launch configuration that automatically links the EC2\-Classic instances to a VPC at launch\. Then, terminate the running instances and let the Auto Scaling group launch new instances that are linked to the VPC\.

### Link to a VPC \(console\)<a name="as-ClassicLink-console"></a>

Use the following procedure to create a launch configuration that links EC2\-Classic instances to the specified VPC and update an existing Auto Scaling group to use the launch configuration\.

**To link EC2\-Classic instances in an Auto Scaling group to a VPC \(old console\)**

1. Verify that the VPC has ClassicLink enabled\. For more information, see [Viewing your ClassicLink\-enabled VPCs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html#classiclink-describe-vpcs-instances) in the *Amazon EC2 User Guide for Linux Instances*\.

1. Create a security group for the VPC that you are going to link EC2\-Classic instances to\. Add rules to control communication between the linked EC2\-Classic instances and instances in the VPC\.

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\. 

1. Choose **Create launch configuration**\.

1. On the **Choose AMI** page, select an AMI\.

1. On the **Choose an Instance Type** page, select an instance type, and then choose **Next: Configure details**\.

1. On the **Configure details** page, do the following:

   1. Enter a name for your launch configuration\.

   1. Expand **Advanced Details**, select the **IP Address Type** that you need, and then select **Link to VPC**\.

   1. For **VPC**, select the VPC with ClassicLink enabled from step 1\.

   1. For **Security Groups**, select the security group from step 2\.

   1. Choose **Skip to review**\.

1. On the **Review** page, make any changes that you need, and then choose **Create launch configuration**\. For **Select an existing key pair or create a new key pair**, select an option, select the acknowledgment check box \(if present\), and then choose **Create launch configuration**\.

1. When prompted, follow the directions to create an Auto Scaling group that uses the new launch configuration\. Be sure to select **Launch into EC2\-Classic** for **Network**\. Otherwise, choose **Cancel** and then add your launch configuration to an existing Auto Scaling group as follows:

   1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

   1. Select the check box next to your Auto Scaling group\.

      A split pane opens up in the bottom part of the page, showing information about the group that's selected\. 

   1. Choose **Actions**, **Edit**\.

   1. For **Launch Configuration**, select your new launch configuration and then choose **Save**\.

### Link to a VPC \(AWS CLI\)<a name="as-ClassicLink-cli"></a>

Use the following procedure to create a launch configuration that links EC2\-Classic instances to the specified VPC and update an existing Auto Scaling group to use the launch configuration\.

**To link EC2\-Classic instances in an Auto Scaling group to a VPC**

1. Verify that the VPC has ClassicLink enabled\. For more information, see [Viewing your ClassicLink\-enabled VPCs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html#classiclink-describe-vpcs-instances) in the *Amazon EC2 User Guide for Linux Instances*\.

1. Create a security group for the VPC that you are going to link EC2\-Classic instances to\. Add rules to control communication between the linked EC2\-Classic instances and instances in the VPC\.

1. Create a launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command as follows\. Specify a value for *vpd\_id* as the ID of the VPC with ClassicLink enabled from step 1 and for *group\_id* as the security group from step 2\.

   ```
   aws autoscaling create-launch-configuration --launch-configuration-name classiclink-config \
     --image-id ami_id --instance-type instance_type \
     --classic-link-vpc-id vpc_id --classic-link-vpc-security-groups group_id
   ```

1. Update your existing Auto Scaling group, for example *my\-asg*, with the launch configuration that you created in the previous step\. Any new EC2\-Classic instances launched in this Auto Scaling group are linked EC2\-Classic instances\. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command as follows\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --launch-configuration-name classiclink-config
   ```

   Alternatively, you can use this launch configuration with a new Auto Scaling group that you create using [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html)\.

## More resources for learning about VPCs<a name="auto-scaling-resources-about-vpcs"></a>

Use the following topics to learn more about VPCs and subnets\.
+ Private Subnets in a VPC
  + [VPC with public and private subnets \(NAT\)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)
  + [NAT instances](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html)
  + [High availability for Amazon VPC NAT instances: An example](https://aws.amazon.com/articles/2781451301784570)
+ Public Subnets in a VPC
  + [VPC with a single public subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html)
+ General VPC Information
  + [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
  + [VPC peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html)
  + [Elastic network interfaces](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html)
  + [Securely connect to Linux instances running in a private VPC](https://blogs.aws.amazon.com/security/post/Tx3N8GFK85UN1G6/Securely-connect-to-Linux-instances-running-in-a-private-Amazon-VPC)