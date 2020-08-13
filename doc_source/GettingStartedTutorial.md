# Getting started with Amazon EC2 Auto Scaling<a name="GettingStartedTutorial"></a>

When you use Amazon EC2 Auto Scaling, you must use certain building blocks to get started\. This tutorial walks you through the process for setting up building blocks to create a basic infrastructure for Amazon EC2 Auto Scaling\. 

Before you create an Auto Scaling group for use with your application, review your application thoroughly as it runs in the AWS Cloud\. Consider the following: 
+ How many Availability Zones the Auto Scaling group should span\.
+ What existing resources can be used, such as security groups or Amazon Machine Images \(AMIs\)\.
+ Whether you want to scale to increase or decrease capacity, or if you just want to ensure that a specific number of servers are always running\. Keep in mind that Amazon EC2 Auto Scaling can do both simultaneously\.
+ What metrics have the most relevance to your application's performance\.
+ How long it takes to launch and configure a server\.

The better you understand your application, the more effective you can make your Auto Scaling architecture\.

The following instructions:
+ Create a configuration template that defines your EC2 instances\. You can choose either the launch template or the launch configuration instructions, based on your preference\. 
+ Create an Auto Scaling group to maintain a fixed number of instances even if an instance becomes unhealthy\.
+ Optionally, delete this basic infrastructure\.

This tutorial assumes that you are familiar with launching EC2 instances and that you have already created a key pair and a security group\. For more information, see [Setting up with Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

To get started, you can launch a single [free tier](https://aws.amazon.com/free/) eligible Linux instance\. If you created your AWS account less than 12 months ago, and have not already exceeded the free tier benefits for Amazon EC2, it will not cost you anything to complete this tutorial, because we help you select options that are within the free tier benefits\. Otherwise, when you follow this tutorial, you incur the standard Amazon EC2 usage fees from the time that the instance launches until you delete the Auto Scaling group \(which is the final task of this tutorial\) and the instance status changes to `terminated`\. 

**Topics**
+ [Step 1: Create a launch template](#gs-create-lt)
+ [Step 2: Create an Auto Scaling group](#gs-create-asg)
+ [Step 3: Verify your Auto Scaling group](#gs-verify-asg)
+ [Step 4: Next steps](#gs-tutorial-next-steps)
+ [Step 5: \(Optional\) Delete your scaling infrastructure](#gs-delete-asg)

## Step 1: Create a launch template<a name="gs-create-lt"></a>

For this step, you create a launch template that specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. Include information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, the key pair, and security groups\. 

For detailed instructions for creating launch templates, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.

**Note**  
Alternatively, you can use a launch configuration to create an Auto Scaling group instead of using a launch template\. For the launch configuration instructions, see [Create a launch configuration](#id-gs-create-lc)\.

**To create a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select an AWS Region\. The Amazon EC2 Auto Scaling resources that you create are tied to the Region that you specify\. 

1. On the navigation pane, under **INSTANCES**, choose **Launch Templates**\.

1. Choose **Create launch template**\.  
![\[Launch templates welcome screen\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-gs-lt-welcome-screen.png)

1. Choose **Create a new template**\. For **Launch template name**, enter a name \(for example, `my_template`\)\. 

1. For **Template version description**, enter a description of the launch template to help you remember what this template is used for \(for example, `test launch template for an Auto Scaling group`\)\. 

1. For **AMI ID**, choose a version of Amazon Linux 2 \(HVM\) from the **Quick Start** list\. The Amazon Machine Image \(AMI\) serves as a basic configuration template for your instances\. 

1. For **Instance type**, choose a hardware configuration that is compatible with the AMI that you specified\. Note that the free tier Linux server is a `t2.micro` instance\.
**Note**  
If your account is less than 12 months old, you can use a `t2.micro` instance for free within certain usage limits\. For more information, see [AWS free tier](https://aws.amazon.com/free/)\.

1. \(Optional\) For **Key pair name**, choose an existing key pair\. You use key pairs to connect to an Amazon EC2 instance with SSH\. Connecting to an instance is not included as part of this tutorial\. Therefore, you don't need to specify a key pair unless you intend to connect to your instance\. 

1. Leave **Network type** set to **VPC**\. 

1. For **Security Groups**, specify the security group in the same VPC that you plan to use as the VPC for your Auto Scaling group\. If you don't specify a security group, your instance is automatically associated with the default security group for the VPC\.

1. You can leave **Network Interfaces** empty\. Leaving the setting empty creates a primary network interface with IP addresses that we select for your instance \(based on the subnet to which the network interface is established\)\. If instead you choose to specify a network interface, the security group must be a part of it\.

1. Scroll down and choose **Create launch template**\.

1. On the confirmation page, choose **Create Auto Scaling group**\.

If you are not currently using launch templates and prefer not to create one now, you can create a launch configuration instead\.

A launch configuration is similar to a launch template, in that it specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. You create the launch configuration by including information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, the key pair, and security groups\. 

**To create a launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, select an AWS Region\. The Auto Scaling resources that you create are tied to the Region that you specify\. 

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. On the **Welcome to Auto Scaling** page, choose **Create Auto Scaling group**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-console-first-time-user-screen.png)

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration, Create a new launch configuration**, and then choose **Next Step**\.

1. For the **Choose AMI** step, there is a list of basic configurations, called Amazon Machine Images \(AMIs\), that serve as templates for your instances\. Choose **Select** for the Amazon Linux 2 AMI\. 

1. For the **Choose Instance Type** step, select a hardware configuration for your instance\. We recommend that you keep the default, a `t2.micro` instance\. Choose **Next: Configure details**\.
**Note**  
If your account is less than 12 months old, you can use a `t2.micro` instance for free within certain usage limits\. For more information, see [AWS free tier](https://aws.amazon.com/free/)\.

1. For the **Configure details** step, do the following:

   1. For **Name**, enter a name for your launch configuration \(for example, `my-first-lc`\)\.

   1. For **Advanced Details**, choose an IP address type\. To provide internet connectivity to instances in a VPC, choose an option that assigns a public IP address\. If an instance is launched into a default VPC, the default is to assign a public IP address\. If you want to provide internet connectivity to your instance but aren't sure whether you have a default VPC, choose **Assign a public IP address to every instance**\.

   1. Choose **Skip to review**\.

1. For the **Review** step, choose **Edit security groups**\. Follow the instructions to choose an existing security group, and then choose **Review**\.
**Note**  
If you have not already created a security group, the wizard automatically defines the `AutoScaling-Security-Group-x` security group to allow you to connect to your instances\. The `AutoScaling-Security-Group-x` security group enables all IP addresses \(0\.0\.0\.0/0\) and allows inbound traffic on port 22 \(SSH\) for Linux instances or port 3389 \(RDP\) for Windows instances depending on the AMI you specified\.

1. For the **Review** step, choose **Create launch configuration**\.

1. Complete the **Select an existing key pair or create a new key pair** step as instructed\. Connecting to an instance is not included as part of this tutorial\. Therefore, you can select **Proceed without a key pair** unless you intend to connect to your instance\.

1. Choose **Create launch configuration**\. 

1. Choose **Create an Auto Scaling group using this launch configuration**\. The wizard to create an Auto Scaling group is displayed\. 

## Step 2: Create an Auto Scaling group<a name="gs-create-asg"></a>

An Auto Scaling group is a collection of EC2 instances, and is the core of Amazon EC2 Auto Scaling\. When you create an Auto Scaling group, you include information such as the subnets for the instances and the initial number of instances to start with\. 

Use the following procedure to continue where you left off after creating either a launch template or a launch configuration\. 

**Note**  
Amazon EC2 Auto Scaling has changed the user interface\. By default, you're shown the new user interface, but you can choose to return to the old user interface\. This topic contains steps for each\. 

**To create an Auto Scaling group \(new console\)**

1. From the new console, on the **Auto Scaling groups** page, choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group\.

1. For **Launch template**, choose one of the following options:

   1. \[Launch template only\] Choose the launch template that you created and then choose the default version of the launch template to use when scaling out\. 

   1. \[Launch configuration only\] Choose **Switch to launch configuration** and choose the launch configuration that you created\.

1. Choose **Next**\. 

   The **Configure settings** page appears, allowing you to configure network settings and giving you options for launching On\-Demand and Spot Instances across multiple instance types \(if you chose a launch template\)\.

1. \[Launch template only\] Keep **Purchase options and instance types** set to **Adhere to the launch template** to quickly create and configure an Auto Scaling group\. 

1. Keep **Network** set to the default VPC for your chosen AWS Region, or select your own VPC\. The default VPC is automatically configured to provide internet connectivity to your instance\. This VPC includes a public subnet in each Availability Zone in the Region\. 

1. For **Subnet**, choose a subnet from each Availability Zone that you want to include\. Use subnets in multiple Availability Zones for high availability\. 

1. Keep the rest of the defaults for this tutorial and choose **Skip to review**\. 
**Note**  
The initial size of the group is determined by its desired capacity\. The default value is `1` instance\. 

1. On the **Review** page, review the information for the group, and then choose **Create Auto Scaling group**\.

We recommend the new user interface, but you can still temporarily use the old user interface\.

**To create an Auto Scaling group \(old console\)**

1. For the **Configure Auto Scaling group details** step, do the following:

   1.  For **Group name**, enter a name for your Auto Scaling group \(for example, `my-first-asg`\)\.   
![\[Auto Scaling group creation screen\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-gs-asg.png)

   1. \[Launch template only\] For **Launch template version**, choose the default version of the launch template to use when scaling out\. 

   1. \[Launch template only\] For **Fleet Composition**, choose **Adhere to the launch template**\. 

   1. Keep **Group size** set to the default value of `1` instance for this tutorial\.

   1. Keep **Network** set to the default VPC for your chosen AWS Region, or select your own VPC\. The default VPC is automatically configured to provide internet connectivity to your instance\. This VPC includes a public subnet in each Availability Zone in the Region\. 

   1. For **Subnet**, choose a subnet from each Availability Zone that you want to include\. Use subnets in multiple Availability Zones for high availability\. 

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, select **Keep this group at its initial size** and choose **Review**\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

## Step 3: Verify your Auto Scaling group<a name="gs-verify-asg"></a>

Now that you have created an Auto Scaling group, you are ready to verify that the group has launched an EC2 instance\.

On the **Auto Scaling groups** page, complete the following procedure\. 

**To verify that your Auto Scaling group has launched an EC2 instance**

1. Select the check box next to the Auto Scaling group that you just created\. 

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group\. The first tab available is the **Details** tab, showing information about the Auto Scaling group\.

1. Choose the second tab, **Activity**\. Under **Activity history**, you can view the progress of activities that are associated with the Auto Scaling group\. The **Status** column shows the current status of your instance\. While your instance is launching, the status column shows `PreInService`\. The status changes to `Successful` after the instance is launched\. You can also use the refresh button to see the current status of your instance\. \(Old console: The **Activity History** tab is where you can view the status of the instance\. While your instance is launching, the status column shows `In progress`\.\) 

1. On the **Instance management** tab, under **Instances**, you can view the status of the instance\. \(Old console: The **Instances** tab is where you can see this\.\)

1. Verify that your instance launched successfully\. It takes a short time for an instance to launch\. 

   The **Lifecycle** column shows the state of your instance\. Initially, your instance is in the `Pending` state\. After an instance is ready to receive traffic, its state is `InService`\.

   The **Health status** column shows the result of the EC2 instance health check on your instance\.

### \(Optional\) Terminate an instance in your Auto Scaling group<a name="gs-asg-terminate-instance"></a>

You can use these steps to learn more about how Amazon EC2 Auto Scaling works, specifically, how it launches new instances when necessary\. The minimum size for the Auto Scaling group that you created in this tutorial is one instance\. Therefore, if you terminate that running instance, Amazon EC2 Auto Scaling must launch a new instance to replace it\.

1. On the **Instance management** tab, under **Instances**, select the ID of the instance\. \(Old console: The **Instances** tab is where you can do this\.\)

   This takes you to the **Instances** page in the Amazon EC2 console, where you can terminate the instance\.

1. Choose **Actions**, **Instance State**, **Terminate**\. When prompted for confirmation, choose **Yes, Terminate**\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\. Select your Auto Scaling group and choose the **Activity** tab\. \(Old console: **Activity History** tab\)\. 

   The default cooldown for the Auto Scaling group is 300 seconds \(5 minutes\), so it takes about 5 minutes until you see the scaling activity\. In the activity history, when the scaling activity starts, you see an entry for the termination of the first instance and an entry for the launch of a new instance\. 

1. On the **Instance management** tab, the **Instances** section shows the new instance only\. 

1. On the navigation pane, under **INSTANCES**, choose **Instances**\. This page shows both the terminated instance and the new running instance\.

## Step 4: Next steps<a name="gs-tutorial-next-steps"></a>

Go to the next step if you would like to delete the basic infrastructure for automatic scaling that you just created\. Otherwise, you can use this infrastructure as your base and try one or more of the following:
+ Manually scale your Auto Scaling group\. For more information, see [Setting capacity limits](asg-capacity-limits.md) and [Manual scaling](as-manual-scaling.md)\.
+ Learn how to automatically scale in response to changes in resource utilization\. If the load increases, your Auto Scaling group can scale out \(add instances\) to handle the demand\. For more information, see [Target tracking scaling policies](as-scaling-target-tracking.md)\.
+ Configure an SNS notification to notify you whenever your Auto Scaling group scales\. For more information, see [Monitoring with Amazon SNS notifications](ASGettingNotifications.md)\.

## Step 5: \(Optional\) Delete your scaling infrastructure<a name="gs-delete-asg"></a>

You can either delete your scaling infrastructure or delete just your Auto Scaling group and keep your launch template or configuration to use later\.

If you launched an instance that is not within the [AWS Free Tier](https://aws.amazon.com/free/), you should terminate your instances to prevent additional charges\. When you terminate the instance, the data associated with it will also be deleted\.

**To delete your Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

1. Choose **Delete**\. \(Old console: Choose **Actions**, **Delete**\.\) When prompted for confirmation, choose **Delete**\.

   A loading icon in the **Name** column indicates that the Auto Scaling group is being deleted\. When the deletion has occurred, the **Desired**, **Min**, and **Max** columns show `0` instances for the Auto Scaling group\. It takes a few minutes to terminate the instance and delete the group\. Refresh the list to see the current state\. 

Skip the following procedure if you would like to keep your launch template\.

**To delete your launch template**

1. On the navigation pane, under **INSTANCES**, choose **Launch Templates**\.

1. Select your launch template\.

1. Choose **Actions**, **Delete template**\. When prompted for confirmation, choose **Delete launch template**\.

Skip the following procedure if you would like to keep your launch configuration\.

**To delete your launch configuration**

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\.

1. Select your launch configuration\.

1. Choose **Actions**, **Delete launch configuration**\. When prompted for confirmation, choose **Yes, Delete**\.