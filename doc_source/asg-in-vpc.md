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

By default, all instances in the VPC run as shared tenancy instances\. Amazon EC2 Auto Scaling also supports Dedicated Instances and Dedicated Hosts\. However, support for Dedicate Hosts is only available for Auto Scaling groups that use a launch template\. For more information, see [Configuring instance tenancy with Amazon EC2 Auto Scaling](auto-scaling-dedicated-instances.md)\.

## Linking EC2\-Classic instances to a VPC<a name="as-ClassicLink"></a>

If you are launching the instances in your Auto Scaling group in EC2\-Classic, you can link them to a VPC using *ClassicLink*\. ClassicLink enables you to associate one or more security groups for the VPC with the EC2\-Classic instances in your Auto Scaling group\. It enables communication between these linked EC2\-Classic instances and instances in the VPC using private IP addresses\. For more information, see [ClassicLink](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html) in the *Amazon EC2 User Guide for Linux Instances*\.

If you have running EC2\-Classic instances in your Auto Scaling group, you can link them to a VPC with ClassicLink enabled\. For more information, see [Linking an instance to a VPC](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-classiclink.html#classiclink-link-instance) in the *Amazon EC2 User Guide for Linux Instances*\. Alternatively, you can update the Auto Scaling group to use a launch configuration that automatically links the EC2\-Classic instances to a VPC at launch\. Then, terminate the running instances and let the Auto Scaling group launch new instances that are linked to the VPC\.

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
+ Private subnets in a VPC
  + [VPC with public and private subnets \(NAT\)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)
  + [NAT instances](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html)
  + [High availability for Amazon VPC NAT instances: An example](https://aws.amazon.com/articles/2781451301784570)
+ Public subnets in a VPC
  + [VPC with a single public subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html)
+ General VPC information
  + [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
  + [VPC peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html)
  + [Elastic network interfaces](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html)
  + [Securely connect to Linux instances running in a private VPC](https://blogs.aws.amazon.com/security/post/Tx3N8GFK85UN1G6/Securely-connect-to-Linux-instances-running-in-a-private-Amazon-VPC)