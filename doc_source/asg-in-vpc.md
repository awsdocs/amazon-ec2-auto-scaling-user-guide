# Launching Auto Scaling Instances in a VPC<a name="asg-in-vpc"></a>

Amazon Virtual Private Cloud \(Amazon VPC\) enables you to define a virtual networking environment in a private, isolated section of the AWS cloud\. You have complete control over your virtual networking environment\. For more information, see the *[Amazon VPC User Guide](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/)*\.

Within a virtual private cloud \(VPC\), you can launch AWS resources such as an Auto Scaling group\. An Auto Scaling group in a VPC works essentially the same way as it does on Amazon EC2 and supports the same set of features\.

A subnet in Amazon VPC is a subdivision within an Availability Zone defined by a segment of the IP address range of the VPC\. Using subnets, you can group your instances based on your security and operational needs\. A subnet resides entirely within the Availability Zone it was created in\. You launch Auto Scaling instances within the subnets\.

To enable communication between the Internet and the instances in your subnets, you must create an Internet gateway and attach it to your VPC\. An Internet gateway enables your resources within the subnets to connect to the Internet through the Amazon EC2 network edge\. If a subnet's traffic is routed to an Internet gateway, the subnet is known as a *public* subnet\. If a subnet's traffic is not routed to an Internet gateway, the subnet is known as a *private* subnet\. Use a public subnet for resources that must be connected to the Internet, and a private subnet for resources that need not be connected to the Internet\.

**Prerequisites**  
Before you can launch your Auto Scaling instances in a VPC, you must first create your VPC environment\. After you create your VPC and subnets, you launch Auto Scaling instances within the subnets\. The easiest way to create a VPC with one public subnet is to use the VPC wizard\. For more information, see the [Amazon VPC Getting Started Guide](http://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/)\.

**Topics**
+ [Default VPC](#as-defaultVPC)
+ [IP Addressing in a VPC](#as-vpc-ipaddress)
+ [Instance Placement Tenancy](#as-vpc-tenancy)
+ [Linking EC2\-Classic Instances to a VPC](#as-ClassicLink)
+ [Examples](#as-vpc-examples)

## Default VPC<a name="as-defaultVPC"></a>

If you have created your AWS account after 2013\-12\-04 or you are creating your Auto Scaling group in a new region, we create a default VPC for you\. Your default VPC comes with a default subnet in each Availability Zone\. If you have a default VPC, your Auto Scaling group is created in the default VPC by default\.

For information about default VPCs and checking whether your account comes with a default VPC, see [Your Default VPC and Subnets](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/default-vpc.html#launching-into) in the *Amazon VPC Developer Guide*\. 

## IP Addressing in a VPC<a name="as-vpc-ipaddress"></a>

When you launch your Auto Scaling instances in a VPC, your instances are automatically assigned a private IP address in the address range of the subnet\. This enables your instances to communicate with other instances in the VPC\.

You can configure your launch configuration to assign public IP addresses to your instances\. Assigning public IP addresses to your instances enables them to communicate with the Internet or other services in AWS\.

When you enable public IP addresses for your instances, they receive both IPv4 and IPv6 addresses if you launch them into a subnet that is configured to automatically assign IPv6 addresses to instances\. Otherwise, they receive IPv4 addresses\. For more information, see [IPv6 Addresses](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#ipv6-addressing) in the *Amazon EC2 User Guide for Linux Instances*\.

## Instance Placement Tenancy<a name="as-vpc-tenancy"></a>

Dedicated Instances are physically isolated at the host hardware level from instances that aren't dedicated and from instances that belong to other AWS accounts\. When you create a VPC, by default its tenancy attribute is set to `default`\. In such a VPC, you can launch instances with a tenancy value of `dedicated` so that they run as single\-tenancy instances\. Otherwise, they run as shared\-tenancy instances by default\. If you set the tenancy attribute of a VPC to `dedicated`, all instances launched in the VPC run as single\-tenancy instances\. For more information, see [Dedicated Instances](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/dedicated-instance.html) in the *Amazon VPC User Guide*\. For pricing information, see the [Amazon EC2 Dedicated Instances](https://aws.amazon.com/ec2/purchasing-options/dedicated-instances/) product page\.

When you create a launch configuration, the default value for the instance placement tenancy is `null` and the instance tenancy is controlled by the tenancy attribute of the VPC\. The following table summarizes the instance placement tenancy of the Auto Scaling instances launched in a VPC\.


| Launch Configuration Tenancy | VPC Tenancy = default | VPC Tenancy = dedicated | 
| --- | --- | --- | 
|  not specified  |  shared\-tenancy instance  |  Dedicated Instance  | 
|  `default`  |  shared\-tenancy instance  |  Dedicated Instance  | 
|  `dedicated`  |  Dedicated Instance  |  Dedicated Instance  | 

You can specify the instance placement tenancy for your launch configuration as `default` or `dedicated` using the [create\-launch\-configuration](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command with the `--placement-tenancy` option\. For example, the following command sets the launch configuration tenancy to `dedicated`:

```
aws autoscaling create-launch-configuration --launch-configuration-name my-launch-config --placement-tenancy dedicated --image-id ...
```

You can use the following [describe\-launch\-configurations](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to verify the instance placement tenancy of the launch configuration:

```
aws autoscaling describe-launch-configurations --launch-configuration-names my-launch-config
```

The following is example output for a launch configuration that creates Dedicated Instances\. Note that `PlacementTenancy` is not part of the output for this command unless you have explicitly set the instance placement tenancy\.

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

## Linking EC2\-Classic Instances to a VPC<a name="as-ClassicLink"></a>

If you are launching the instances in your Auto Scaling group in EC2\-Classic, you can link them to a VPC using *ClassicLink*\. ClassicLink enables you to associate one or more security groups for the VPC with the EC2\-Classic instances in your Auto Scaling group, enabling communication between these linked EC2\-Classic instances and instances in the VPC using private IP addresses\. For more information, see [ClassicLink](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html) in the *Amazon EC2 User Guide for Linux Instances*\.

If you have running EC2\-Classic instances in your Auto Scaling group, you can link them to a VPC with ClassicLink enabled\. For more information, see [Linking an Instance to a VPC](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html#classiclink-link-instance) in the *Amazon EC2 User Guide for Linux Instances*\. Alternatively, you can update the Auto Scaling group to use a launch configuration that automatically links the EC2\-Classic instances to a VPC at launch, then terminate the running instances and let the Auto Scaling group launch new instances that are linked to the VPC\.

### Link to a VPC Using the AWS Management Console<a name="as-ClassicLink-console"></a>

Use the following procedure to create a launch configuration that links EC2\-Classic instances to the specified VPC and update an existing Auto Scaling group to use the launch configuration\.

**To link EC2\-Classic instances in an Auto Scaling group to a VPC using the console**

1. Verify that the VPC has ClassicLink enabled\. For more information, see [Viewing Your ClassicLink\-Enabled VPCs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html#classiclink-describe-vpcs-instances) in the *Amazon EC2 User Guide for Linux Instances*\.

1. Create a security group for the VPC that you are going to link EC2\-Classic instances to, with rules to control communication between the linked EC2\-Classic instances and instances in the VPC\.

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Launch Configurations**\. If you are new to Amazon EC2 Auto Scaling, you see a welcome page\. Choose **Create Auto Scaling group**\.

1. Choose **Create launch configuration**\.

1. On the **Choose AMI** page, select an AMI\.

1. On the **Choose an Instance Type** page, select an instance type, and then choose **Next: Configure details**\.

1. On the **Configure details** page, do the following:

   1. Type a name for your launch configuration\.

   1. Expand **Advanced Details**, select the **IP Address Type** that you need, and then select **Link to VPC**\.

   1. For **VPC**, select the VPC with ClassicLink enabled from step 1\.

   1. For **Security Groups**, select the security group from step 2\.

   1. Choose **Skip to review**\.

1. On the **Review** page, make any changes that you need, and then choose **Create launch configuration**\. For **Select an existing key pair or create a new key pair**, select an option, select the acknowledgment check box \(if present\), and then choose **Create launch configuration**\.

1. When prompted, follow the directions to create an Auto Scaling group that uses the new launch configuration\. Be sure to select **Launch into EC2\-Classic** for **Network**\. Otherwise, choose **Cancel** and then add your launch configuration to an existing Auto Scaling group as follows:

   1. On the navigation pane, choose **Auto Scaling Groups**\.

   1. Select your Auto Scaling group, choose **Actions**, **Edit**\.

   1. For **Launch Configuration**, select your new launch configuration and then choose **Save**\.

### Link to a VPC Using the AWS CLI<a name="as-ClassicLink-cli"></a>

Use the following procedure to create a launch configuration that links EC2\-Classic instances to the specified VPC and update an existing Auto Scaling group to use the launch configuration\.

**To link EC2\-Classic instances in an Auto Scaling group to a VPC using the AWS CLI**

1. Verify that the VPC has ClassicLink enabled\. For more information, see [Viewing Your ClassicLink\-Enabled VPCs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html#classiclink-describe-vpcs-instances) in the *Amazon EC2 User Guide for Linux Instances*\.

1. Create a security group for the VPC that you are going to link EC2\-Classic instances to, with rules to control communication between the linked EC2\-Classic instances and instances in the VPC\.

1. Create a launch configuration using the [create\-launch\-configuration](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command as follows, where *vpd\_id* is the ID of the VPC with ClassicLink enabled from step 1 and *group\_id* is the security group from step 2:

   ```
   aws autoscaling create-launch-configuration --launch-configuration-name classiclink-config 
   --image-id ami_id --instance-type instance_type
   --classic-link-vpc-id vpc_id --classic-link-vpc-security-groups group_id
   ```

1. Update your existing Auto Scaling group, for example *my\-asg*, with the launch configuration that you created in the previous step\. Any new EC2\-Classic instances launched in this Auto Scaling group are linked EC2\-Classic instances\. Use the [update\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command as follows:

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg 
   --launch-configuration-name classiclink-config
   ```

   Alternatively, you can use this launch configuration with a new Auto Scaling group that you create using [create\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html)\.

## Examples<a name="as-vpc-examples"></a>

For examples, see the following tutorials:
+ [Getting Started with Amazon EC2 Auto Scaling](GettingStartedTutorial.md)
+ [Hosting a Web App on Amazon Web Services](http://docs.aws.amazon.com/gettingstarted/latest/wah-linux/web-app-hosting-intro.html)
+ [Hosting a \.NET Web App on Amazon Web Services](http://docs.aws.amazon.com/gettingstarted/latest/wah/web-app-hosting-intro.html)