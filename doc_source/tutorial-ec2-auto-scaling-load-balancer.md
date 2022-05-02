# Tutorial: Set up a scaled and load\-balanced application<a name="tutorial-ec2-auto-scaling-load-balancer"></a>

**Important**  
Before you explore this tutorial, we recommend that you first review the following introductory tutorial: [Get started with Amazon EC2 Auto Scaling](get-started-with-ec2-auto-scaling.md)\.

Registering your Auto Scaling group with an Elastic Load Balancing load balancer helps you set up a load\-balanced application\. Elastic Load Balancing works with Amazon EC2 Auto Scaling to distribute incoming traffic across your healthy Amazon EC2 instances\. This increases the scalability and availability of your application\. You can enable Elastic Load Balancing within multiple Availability Zones to increase the fault tolerance of your applications\. 

In this tutorial, we cover the basics steps for setting up a load\-balanced application when the Auto Scaling group is created\. When complete, your architecture should look similar to the following diagram:

![\[An Auto Scaling group with an Application Load Balancer.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/elb-tutorial-architecture-diagram.png)

Elastic Load Balancing supports different types of load balancers\. We recommend that you use an Application Load Balancer for this tutorial\. 

For more information about introducing a load balancer into your architecture, see [Use Elastic Load Balancing to distribute traffic across the instances in your Auto Scaling group](autoscaling-load-balancer.md)\.

**Topics**
+ [Prerequisites](#as-register-lbs-prerequisites)
+ [Step 1: Set up a launch template or launch configuration](#as-register-lbs-create-lt-console)
+ [Step 2: Create an Auto Scaling group](#as-register-lbs-create-asg-console)
+ [Step 3: Verify that your load balancer is attached](#as-register-lbs-verify-console)
+ [Step 4: Next steps](#as-lbs-app-next-steps)
+ [Step 5: Clean up](#as-lbs-app-clean-up)

## Prerequisites<a name="as-register-lbs-prerequisites"></a>
+ A load balancer and target group\. Make sure to choose the same Availability Zones for the load balancer that you plan to use for your Auto Scaling group\. For more information, see [Getting started with Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-getting-started.html) in the *Elastic Load Balancing User Guide*\.
+ A security group for your launch template or launch configuration\. The security group must allow access from the load balancer on both the listener port \(usually port 80 for HTTP traffic\) and the port that you want Elastic Load Balancing to use for health checks\. For more information, see the applicable documentation:
  + [Target security groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-register-targets.html#target-security-groups) in the *User Guide for Application Load Balancers*
  + [Target security groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#target-security-groups) in the *User Guide for Network Load Balancers*

  Optionally, if your instances will have public IP addresses, you can allow SSH traffic for connecting to the instances\. 
+ \(Optional\) An IAM role that grants your application access to AWS\.
+ \(Optional\) An Amazon Machine Image \(AMI\) defined as the source template for your Amazon EC2 instances\. To create one now, launch an instance\. Specify the IAM role \(if you created one\) and any configuration scripts that you need as user data\. Connect to the instance and customize it\. For example, you can install software and applications, copy data, and attach additional EBS volumes\. Test your applications on your instance to ensure that it is configured correctly\. Save this updated configuration as a custom AMI\. If you don't need the instance later, you can terminate it\. Instances launched from this new custom AMI include the customizations that you made when you created the AMI\. 
+ A virtual private cloud \(VPC\)\. This tutorial refers to the default VPC, but you can use your own\. If using your own VPC, make sure that it has a subnet mapped to each Availability Zone of the Region you are working in\. At minimum, you must have two public subnets available to create the load balancer\. You must also have either two private subnets or two public subnets to create your Auto Scaling group and register it with the load balancer\.

## Step 1: Set up a launch template or launch configuration<a name="as-register-lbs-create-lt-console"></a>

Use either a launch template or a launch configuration for this tutorial\. 

If you already have a launch template that you'd like to use, select it by using the following procedure\.

**Note**  
Alternatively, you can use a launch configuration instead of a launch template\. For the launch configuration instructions, see [Select or create a launch configuration](#as-register-lbs-create-lc-console)\.

**To select an existing launch template**

1. Open the [Launch templates page](https://console.aws.amazon.com/ec2/v2/#LaunchTemplates) of the Amazon EC2 console\.

1. On the navigation bar at the top of the screen, choose the Region where the load balancer was created\.

1. Select a launch template\.

1. Choose **Actions**, **Create Auto Scaling group**\.

Alternatively, to create a new launch template, use the following procedure\.

**To create a launch template**

1. Open the [Launch templates page](https://console.aws.amazon.com/ec2/v2/#LaunchTemplates) of the Amazon EC2 console\.

1. On the navigation bar at the top of the screen, choose the Region where the load balancer was created\.

1. Choose **Create launch template**\. 

1. Enter a name and provide a description for the initial version of the launch template\.

1. For **Application and OS Images \(Amazon Machine Image\)**, choose the ID of the AMI for your instances\. You can search through all available AMIs, or from the **Recent** or **Quick Start** list, select an AMI\. If you don't see the AMI that you need, you can choose **Browser more AMIs** to browse the full AMI catalog\.

1. For **Instance type**, select a hardware configuration for your instances that is compatible with the AMI that you specified\.

1. \(Optional\) For **Key pair \(login\)**, choose the key pair to use when connecting to your instances\.

1. For **Network settings**, expand **Advanced network configuration** and do the following:

   1. Choose **Add network interface** to configure the primary network interface\.

   1. \(Optional\) For **Auto\-assign public IP**, keep the default value, **Don't include in launch template**\. When you create your Auto Scaling group, you can assign a public IPv4 address to instances in your Auto Scaling group by using subnets that have the public IPv4 addressing attribute enabled, such as the default subnets in the default VPC\. Alternatively, if you don't need to connect to your instances, you can choose **Disable** to prevent instances in your group from receiving traffic directly from the internet\. In this case, they will receive traffic only from the load balancer\.

   1. For **Security group ID**, specify a security group for your instances from the same VPC as the load balancer\. 

   1. For **Delete on termination**, choose **Yes**\. This deletes the network interface when the Auto Scaling group scales in, and terminates the instance to which the network interface is attached\. 

1. \(Optional\) To securely distribute credentials to your instances, for **Advanced details**, **IAM instance profile**, enter the Amazon Resource Name \(ARN\) of your IAM role\.

1. \(Optional\) To specify user data or a configuration script for your instances, paste it into **Advanced details**, **User data**\.

1. Choose **Create launch template**\. 

1. On the confirmation page, choose **Create Auto Scaling group**\.

### Select or create a launch configuration<a name="as-register-lbs-create-lc-console"></a>

If you already have a launch configuration that you'd like to use, select it by using the following procedure\.

**To select an existing launch configuration**

1. Open the [Launch configurations page](https://console.aws.amazon.com/ec2autoscaling/home?#/lc) of the Amazon EC2 console\.

1. On the navigation bar at the top of the screen, choose the Region where the load balancer was created\.

1. Select a launch configuration\.

1. Choose **Actions**, **Create Auto Scaling group**\.

Alternatively, to create a new launch configuration, use the following procedure\.

**To create a launch configuration**

1. Open the [Launch configurations page](https://console.aws.amazon.com/ec2autoscaling/home?#/lc) of the Amazon EC2 console\.

1. On the navigation bar at the top of the screen, choose the Region where the load balancer was created\.

1. Choose **Create launch configuration**, and enter a name for your launch configuration\. 

1. For **Amazon machine image \(AMI\)**, enter the ID of the AMI for your instances as search criteria\. 

1. For **Instance type**, select a hardware configuration for your instance\.

1. Under **Additional configuration**, pay attention to the following fields:

   1. \(Optional\) To securely distribute credentials to your EC2 instance, for **IAM instance profile**, select your IAM role\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

   1. \(Optional\) To specify user data or a configuration script for your instance, paste it into **Advanced details**, **User data**\.

   1. \(Optional\) For **Advanced details**, **IP address type**, keep the default value\. When you create your Auto Scaling group, you can assign a public IP address to instances in your Auto Scaling group by using subnets that have the public IP addressing attribute enabled, such as the default subnets in the default VPC\. Alternatively, if you don't need to connect to your instances, you can choose **Do not assign a public IP address to any instances** to prevent instances in your group from receiving traffic directly from the internet\. In this case, they will receive traffic only from the load balancer\.

1. For **Security groups**, choose an existing security group from the same VPC as the load balancer\. If you keep the **Create a new security group** option selected, a default SSH rule is configured for Amazon EC2 instances running Linux\. A default RDP rule is configured for Amazon EC2 instances running Windows\. 

1. For **Key pair \(login\)**, choose an option under **Key pair options**\. 

   If you've already configured an Amazon EC2 instance key pair, you can choose it here\. 

   If you don't already have an Amazon EC2 instance key pair, choose **Create a new key pair** and give it a recognizable name\. Choose **Download key pair** to download the key pair to your computer\. 
**Important**  
If you need to connect to your instances, do not choose **Proceed without a key pair**\.

1. Select the acknowledgment check box, and then choose **Create launch configuration**\.

1. Select the check box next to the name of your new launch configuration and choose **Actions**, **Create Auto Scaling group**\. 

## Step 2: Create an Auto Scaling group<a name="as-register-lbs-create-asg-console"></a>

Use the following procedure to continue where you left off after creating or selecting your launch template or launch configuration\. 

**To create an Auto Scaling group**

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group\.

1. \[Launch template only\] For **Launch template**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

1. Choose **Next**\. 

   The **Choose instance launch options** page appears, allowing you to choose the VPC network settings you want the Auto Scaling group to use and giving you options for launching On\-Demand and Spot Instances \(if you chose a launch template\)\. 

1. In the **Network** section, for **VPC**, choose the VPC that you used for your load balancer\. If you chose the default VPC, it is automatically configured to provide internet connectivity to your instances\. This VPC includes a public subnet in each Availability Zone in the Region\. 

1. For **Availability Zones and subnets**, choose one or more subnets from each Availability Zone that you want to include, based on which Availability Zones the load balancer is in\. For more information, see [Considerations when choosing VPC subnets](asg-in-vpc.md#as-vpc-considerations)\.

1. \[Launch template only\] In the **Instance type requirements** section, use the default setting to simplify this step\. \(Do not override the launch template\.\) For this tutorial, you will launch only On\-Demand Instances using the instance type specified in your launch template\.

1. Choose **Next** to go to the **Configure advanced options** page\. 

1. To attach the group to an existing load balancer, in the **Load balancing** section, choose **Attach to an existing load balancer**\. You can choose **Choose from your load balancer target groups** or **Choose from Classic Load Balancers**\. You can then choose the name of a target group for the Application Load Balancer or Network Load Balancer you created, or choose the name of a Classic Load Balancer\.

1. \(Optional\) To use Elastic Load Balancing health checks, for **Health checks**, choose **ELB** under **Health check type**\.

1. When you have finished configuring the Auto Scaling group, choose **Skip to review**\. 

1. On the **Review** page, review the details of your Auto Scaling group\. You can choose **Edit** to make changes\. When you are finished, choose **Create Auto Scaling group**\.

After you have created the Auto Scaling group with the load balancer attached, the load balancer automatically registers new instances as they come online\. You have only one instance at this point, so there isn't much to register\. However, you can add additional instances by updating the desired capacity of the group\. For step\-by\-step instructions, see [Manual scaling](as-manual-scaling.md)\. 

## Step 3: Verify that your load balancer is attached<a name="as-register-lbs-verify-console"></a>

**To verify that your load balancer is attached**

1. From the [Auto Scaling groups page](https://console.aws.amazon.com/ec2autoscaling) of the Amazon EC2 console, select the check box next to your Auto Scaling group\.

1. On the **Details** tab, **Load balancing** shows any attached load balancer target groups or Classic Load Balancers\.

1. On the **Activity** tab, in **Activity history**, you can verify that your instances launched successfully\. The **Status** column shows whether your Auto Scaling group has successfully launched instances\. If your instances fail to launch, you can find troubleshooting ideas for common instance launch issues in [Troubleshoot Amazon EC2 Auto Scaling](CHAP_Troubleshooting.md)\.

1. On the **Instance management** tab, under **Instances**, you can verify that your instances are ready to receive traffic\. Initially, your instances are in the `Pending` state\. After an instance is ready to receive traffic, its state is `InService`\. The **Health status** column shows the result of the Amazon EC2 Auto Scaling health checks on your instances\. Although an instance may be marked as healthy, the load balancer will only send traffic to instances that pass the load balancer health checks\.

1. Verify that your instances are registered with the load balancer\. Open the [Target groups page](https://console.aws.amazon.com/ec2/v2/#TargetGroups) of the Amazon EC2 console\. Select your target group, and then choose the **Targets** tab\. If the state of your instances is `initial`, it's probably because they are still in the process of being registered, or they are still undergoing health checks\. When the state of your instances is `healthy`, they are ready for use\.

## Step 4: Next steps<a name="as-lbs-app-next-steps"></a>

Now that you have completed this tutorial, you can learn more:
+ You can configure your Auto Scaling group to use Elastic Load Balancing health checks\. If you enable load balancer health checks and an instance fails the health checks, the Auto Scaling group considers the instance unhealthy and replaces it\. For more information, see [Add Elastic Load Balancing health checks](as-add-elb-healthcheck.md)\.
+ You can expand your application to an additional Availability Zone in the same Region to increase fault tolerance if there is a service disruption\. For more information, see [Add Availability Zones](as-add-availability-zone.md)\.
+ You can configure your Auto Scaling group to use a target tracking scaling policy\. This automatically increases or decreases the number of instances as the demand on your instances changes\. This allows the group to handle changes in the amount of traffic that your application receives\. For more information, see [Target tracking scaling policies](as-scaling-target-tracking.md)\. 

## Step 5: Clean up<a name="as-lbs-app-clean-up"></a>

After you're finished with the resources that you created for this tutorial, you should consider cleaning them up to avoid incurring unnecessary charges\.

**To delete your Auto Scaling group**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2autoscaling) in the Amazon EC2 console\.

1. Select your Auto Scaling group\.

1. Choose **Delete**\. When prompted for confirmation, choose **Delete**\.

   A loading icon in the **Name** column indicates that the Auto Scaling group is being deleted\. When the deletion has occurred, the **Desired**, **Min**, and **Max** columns show `0` instances for the Auto Scaling group\. It takes a few minutes to terminate the instance and delete the group\. Refresh the list to see the current state\. 

Skip the following procedure if you would like to keep your launch template\.

**To delete your launch template**

1. Open the [Launch templates page](https://console.aws.amazon.com/ec2/v2/#LaunchTemplates) of the Amazon EC2 console\.

1. Select your launch template\.

1. Choose **Actions**, **Delete template**\. When prompted for confirmation, choose **Delete launch template**\.

Skip the following procedure if you would like to keep your launch configuration\.

**To delete your launch configuration**

1. Open the [Launch configurations page](https://console.aws.amazon.com/ec2autoscaling/home?#/lc) of the Amazon EC2 console\.

1. Select your launch configuration\.

1. Choose **Actions**, **Delete launch configuration**\. When prompted for confirmation, choose **Yes, Delete**\.

Skip the following procedure if you want to keep the load balancer for future use\. 

**To delete your load balancer**

1. Open the [Load balancers page](https://console.aws.amazon.com/ec2/v2/#LoadBalancers) of the Amazon EC2 console\.

1. Choose the load balancer and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete your target group**

1. Open the [Target groups page](https://console.aws.amazon.com/ec2/v2/#TargetGroups) of the Amazon EC2 console\.

1. Choose the target group and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes**\.