# Launching Auto Scaling instances in a VPC<a name="asg-in-vpc"></a>

Amazon Virtual Private Cloud \(Amazon VPC\) enables you to define a virtual networking environment in a private, isolated section of the AWS Cloud\. You have complete control over your virtual networking environment\.

Within a virtual private cloud \(VPC\), you can launch AWS resources such as Auto Scaling groups\. An Auto Scaling group in a VPC works essentially the same way as it does on Amazon EC2 and supports the same set of features\.

A subnet in Amazon VPC is a subdivision within an Availability Zone defined by a segment of the IP address range of the VPC\. Using subnets, you can group your instances based on your security and operational needs\. A subnet resides entirely within the Availability Zone it was created in\. You launch Auto Scaling instances within the subnets\.

To enable communication between the internet and the instances in your subnets, you must create an internet gateway and attach it to your VPC\. An internet gateway enables your resources within the subnets to connect to the internet through the Amazon EC2 network edge\. If a subnet's traffic is routed to an internet gateway, the subnet is known as a *public* subnet\. If a subnet's traffic is not routed to an internet gateway, the subnet is known as a *private* subnet\. Use a public subnet for resources that must be connected to the internet, and a private subnet for resources that need not be connected to the internet\. For more information about giving internet access to instances in a VPC, see [Accessing the internet](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html#what-is-connectivity) in the *Amazon VPC User Guide*\.

**Topics**
+ [Default VPC](#as-defaultVPC)
+ [Nondefault VPC](#as-nondefaultVPC)
+ [Considerations when choosing VPC subnets](#as-vpc-considerations)
+ [IP addressing in a VPC](#as-vpc-ipaddress)
+ [Network interfaces in a VPC](#as-vpc-network-interfaces)
+ [Instance placement tenancy](#as-vpc-tenancy)
+ [More resources for learning about VPCs](#auto-scaling-resources-about-vpcs)

## Default VPC<a name="as-defaultVPC"></a>

If you created your AWS account after December 4, 2013 or you are creating your Auto Scaling group in a new AWS Region, we create a default VPC for you\. Your default VPC comes with a default subnet in each Availability Zone\. If you have a default VPC, your Auto Scaling group is created in the default VPC by default\.

If you created your AWS account before December 4, 2013, it may allow you to choose between Amazon VPC and EC2\-Classic in certain Regions\. If you have one of these older accounts, you might have Auto Scaling groups in EC2\-Classic in some Regions instead of Amazon VPC\. Migrating the classic instances to a VPC is highly recommended\. One of the advantages of using a VPC is the ability to put your instances in private subnets with no route to the internet\. 

For information about default VPCs and checking whether your account comes with a default VPC, see [Default VPC and default subnets](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html) in the *Amazon VPC User Guide*\. 

## Nondefault VPC<a name="as-nondefaultVPC"></a>

You can choose to create additional VPCs by going to the Amazon VPC page in the AWS Management Console and selecting **Launch VPC Wizard**\. 

You're presented with the following four options for network architectures:
+ Amazon VPC with a single public subnet only
+ Amazon VPC with public and private subnets
+ Amazon VPC with public and private subnets and hardware VPN access
+ Amazon VPC with a private subnet only and hardware VPN access

For more information, see the *[Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)*\.

## Considerations when choosing VPC subnets<a name="as-vpc-considerations"></a>

Note the following considerations when choosing VPC subnets: 
+ If you're attaching an Elastic Load Balancing load balancer to your Auto Scaling group, the instances can be launched into either public or private subnets\. However, the load balancer can be created in public subnets only\.
+ If you're accessing your Auto Scaling instances directly through SSH, the instances can be launched into public subnets only\. 
+ If you're accessing no\-ingress Auto Scaling instances using AWS Systems Manager Session Manager, the instances can be launched into either public or private subnets\. 
+ If you're using private subnets, you can allow the Auto Scaling instances to access the internet by using a public NAT gateway\.
+ By default, the default subnets in a default VPC are public subnets\. 

## IP addressing in a VPC<a name="as-vpc-ipaddress"></a>

When you launch your Auto Scaling instances in a VPC, your instances are automatically assigned a private IP address from the CIDR range of the subnet in which the instance is launched\. This enables your instances to communicate with other instances in the VPC\.

You can configure a launch template or launch configuration to assign public IP addresses to your instances\. Assigning public IP addresses to your instances enables them to communicate with the internet or other Amazon Web Services\.

When you enable public IP addresses for your instances and launch them into a subnet that is configured to automatically assign IPv6 addresses, they receive both IPv4 and IPv6 addresses\. Otherwise, they receive only IPv4 addresses\. For more information, see [IPv6 addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#ipv6-addressing) in the *Amazon EC2 User Guide for Linux Instances*\.

For information on specifying CIDR ranges for your VPC or subnet, see the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\.

## Network interfaces in a VPC<a name="as-vpc-network-interfaces"></a>

Each instance in your VPC has a default network interface \(the primary network interface\)\. You cannot detach a primary network interface from an instance\. You can create and attach an additional network interface to any instance in your VPC\. The number of network interfaces you can attach varies by instance type\.

When launching an instance using a launch template, you can specify additional network interfaces\. However, launching an Auto Scaling instance with multiple network interfaces automatically creates each interface in the same subnet as the instance\. This is because Amazon EC2 Auto Scaling ignores the subnets defined in the launch template in favor of what is specified in the Auto Scaling group\. For more information, see [Creating a launch template for an Auto Scaling group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html)\.

If you create or attach two or more network interfaces from the same subnet to an instance, you might encounter networking issues such as asymmetric routing, especially on instances using a variant of non\-Amazon Linux\. If you need this type of configuration, you must configure the secondary network interface within the OS\. For an example, see [How can I make my secondary network interface work in my Ubuntu EC2 instance?](http://aws.amazon.com/premiumsupport/knowledge-center/ec2-ubuntu-secondary-network-interface/) in the AWS Knowledge Center\.

## Instance placement tenancy<a name="as-vpc-tenancy"></a>

By default, all instances in the VPC run as shared tenancy instances\. Amazon EC2 Auto Scaling also supports Dedicated Instances and Dedicated Hosts\. However, support for Dedicated Hosts is only available for Auto Scaling groups that use a launch template\. For more information, see [Configuring instance tenancy with a launch configuration](auto-scaling-dedicated-instances.md)\.

## More resources for learning about VPCs<a name="auto-scaling-resources-about-vpcs"></a>

Use the following topics to learn more about VPCs and subnets\.
+ Private subnets in a VPC
  + [VPC with public and private subnets \(NAT\)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)
  + [NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
+ Public subnets in a VPC
  + [VPC with a single public subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html)
+ General VPC information
  + [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
  + [VPC peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html)
  + [Elastic network interfaces](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html)
  + [Migrate from EC2\-Classic to a VPC](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-migrate.html)