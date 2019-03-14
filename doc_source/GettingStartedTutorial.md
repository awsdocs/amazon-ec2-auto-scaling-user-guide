# Getting Started with Amazon EC2 Auto Scaling<a name="GettingStartedTutorial"></a>

When you use Amazon EC2 Auto Scaling, you must use certain building blocks to get started\. This tutorial walks you through the process for setting up the basic infrastructure for Amazon EC2 Auto Scaling\. 

Before you create an Auto Scaling group for use with your application, review your application thoroughly as it runs in the AWS Cloud\. Take note of the following: 
+ How long it takes to launch and configure a server\.
+ What metrics have the most relevance to your application's performance\.
+ How many Availability Zones the Auto Scaling group should span\.
+ What existing resources can be used, such as security groups or Amazon Machine Images \(AMIs\)\.
+ Do you want to scale to increase or decrease capacity, or do you just want to ensure that a specific number of servers are always running? Keep in mind that Amazon EC2 Auto Scaling can do both simultaneously\.

The better you understand your application, the more effective you can make your Auto Scaling architecture\.

The following instructions are for a configuration template that defines your EC2 instances, creates an Auto Scaling group to maintain a fixed number of instances even if an instance becomes unhealthy, and optionally deletes this basic infrastructure\. 

This tutorial assumes that you are familiar with launching EC2 instances and have already created a key pair and a security group\. For more information, see [Setting Up with Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

**Topics**
+ [Step 1: Create a Launch Template](#gs-create-lt)
+ [Step 2: Create an Auto Scaling Group](#gs-create-asg)
+ [Step 3: Verify Your Auto Scaling Group](#gs-verify-asg)
+ [Step 4: \(Optional\) Delete Your Scaling Infrastructure](#gs-delete-asg)

## Step 1: Create a Launch Template<a name="gs-create-lt"></a>

For this step, you create a launch template that specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. Include information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, key pairs, security groups, and block device mappings\.

**To create a launch template for an Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, select a region\. The Amazon EC2 Auto Scaling resources that you create are tied to the region you specify\. 

1. On the navigation pane, choose **Instances**, **Launch Templates**\.

1. Choose **Create launch template**\.  
![\[Launch templates welcome screen\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-gs-lt-welcome-screen.png)

1. Choose **Create a new template**\. Provide a name \(for example, `my_template`\) and description for the launch template\. 

1. For **AMI ID**, choose a version of Amazon Linux 2 \(HVM\) from the **Quick Start** list\. The Amazon Machine Image \(AMI\) serves as basic configuration templates for your instances\. 

1. For **Instance type**, choose a hardware configuration that is compatible with the AMI that you specified\. Note that the free tier Linux server is a `t2.micro` instance\.
**Note**  
If your account is less than 12 months old, you can use a `t2.micro` instance for free within certain usage limits\. For more information, see [AWS Free Tier](https://aws.amazon.com/free/)\.

1. \(Optional\) For **Key pair name**, type the name of the key pair to use when connecting to your instances\.

1. \(Optional\) For **Network type**, choose **VPC**\. 

1. Skip **Security Groups** to configure a security group as part of the network interface\. You cannot specify security groups in both places\.

1. For **Network interfaces**, configure **Auto\-assign public IP**, **Security group ID**, and **Delete on termination**\. To launch instances into a VPC, you must specify a security group that is created for that VPC\. You can leave the other fields empty and this creates a primary network interface with IP addresses that we select for your instances\. 

1. \(Optional\) For **Storage \(Volumes\)**, specify volumes to attach to the instances in addition to the volumes specified by the AMI you specified\.

1. \(Optional\) For **Tags**, specify one or more tags to associate with the instances and volumes\.

1. Choose **Create launch template**\.

1. On the confirmation page, choose **Create Auto Scaling group**\.

If you are not currently using launch templates, you can create a launch configuration instead\.

A launch configuration is similar to a launch template, in that it specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. Create the launch configuration by including information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, the key pair, security groups, and block device mapping\. 

**To create a launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, select a Region\. The Auto Scaling resources that you create are tied to the Region that you specify\. 

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. On the **Welcome to Auto Scaling** page, choose **Create Auto Scaling group**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-console-first-time-user-screen.png)

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration, Create a new launch configuration**, and then choose **Next Step**\.

1. For the **Choose AMI** step, there is a list of basic configurations, called Amazon Machine Images \(AMIs\), that serve as templates for your instances\. Choose **Select** for the Amazon Linux 2 AMI\. 

1. For the **Choose Instance Type** step, select a hardware configuration for your instances\. We recommend that you keep the default, a `t2.micro` instance\. Choose **Next: Configure details**\.

1. For the **Configure details** step, do the following:

   1. For **Name**, type a name for your launch configuration \(for example, `my-first-lc`\)\.

   1. For **Advanced Details**, select an IP address type\. If you want to connect to an instance in a VPC, you must select an option that assigns a public IP address\. If you want to connect to your instance but aren't sure whether you have a default VPC, select **Assign a public IP address to every instance**\.

   1. Choose **Skip to review**\.

1. For the **Review** step, choose **Edit security groups**\. Follow the instructions to choose an existing security group, and then choose **Review**\.

1. For the **Review** step, choose **Create launch configuration**\.

1. Complete the **Select an existing key pair or create a new key pair** step as instructed\. You won't connect to your instance as part of this tutorial\. Therefore, you can select **Proceed without a key pair** unless you intend to connect to your instance\.

1. Choose **Create launch configuration**\. The launch configuration is created and the wizard to create an Auto Scaling group is displayed\. 

## Step 2: Create an Auto Scaling Group<a name="gs-create-asg"></a>

An Auto Scaling group is a collection of EC2 instances, and the core of Amazon EC2 Auto Scaling\. When you create an Auto Scaling group, you include information such as the subnets for the instances and the initial number of instances to start with\. 

Use the following procedure to continue where you left off after creating the launch template\.

**To create an Auto Scaling group using a launch template**

1. For the **Configure Auto Scaling group details** step, do the following:

   1.  For **Group name**, type a name for your Auto Scaling group \(for example, `my-first-asg`\)\.   
![\[Auto Scaling group creation screen\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-gs-asg.png)

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

   1. For **Fleet Composition**, choose **Adhere to the launch template**\. 

   1. Keep **Group size** set to the default value of `1` instance for this tutorial\.

   1. Keep **Network** set to the default VPC for your chosen AWS region, or select your own VPC\. 

   1. For **Subnet**, choose a subnet for the VPC\. 
**Note**  
You can choose the Availability Zone for your instance by choosing its corresponding default subnet\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, select **Keep this group at its initial size** and choose **Review**\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

## Step 3: Verify Your Auto Scaling Group<a name="gs-verify-asg"></a>

Now that you have created your Auto Scaling group, you are ready to verify that the group has launched an EC2 instance\.

**To verify that your Auto Scaling group has launched an EC2 instance**

1. On the **Auto Scaling Groups** page, select the Auto Scaling group that you just created\.

1. The **Details** tab provides information about the Auto Scaling group\.  
![\[Auto Scaling group details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-gs-group-details.png)

1. On the **Activity History** tab, the **Status** column shows the current status of your instance\. While your instance is launching, the status column shows `In progress`\. The status changes to `Successful` after the instance is launched\. You can also use the refresh button to see the current status of your instance\.

1. On the **Instances** tab, the **Lifecycle** column shows the state of your instance\. You can see that your Auto Scaling group has launched your EC2 instance, and that it is in the `InService` lifecycle state\. The **Health Status** column shows the result of the EC2 instance health check on your instance\.  
![\[Auto Scaling group instances\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-gs-group-instances.png)

### \(Optional\) Terminate an Instance in Your Auto Scaling Group<a name="gs-asg-terminate-instance"></a>

 If you want, you can try the following experiment to learn more about Amazon EC2 Auto Scaling\. The minimum size for your Auto Scaling group is one instance\. Therefore, if you terminate the running instance, Amazon EC2 Auto Scaling must launch a new instance to replace it\.

1. On the **Instances** tab, select the ID of the instance\. This shows you the instance on the **Instances** page\.

1. Choose **Actions**, **Instance State**, **Terminate**\. When prompted for confirmation, choose **Yes, Terminate**\.

1. On the navigation pane, choose **Auto Scaling Groups**\. Select your Auto Scaling group and choose the **Activity History** tab\. The default cooldown for the Auto Scaling group is 300 seconds \(5 minutes\), so it takes about 5 minutes until you see the scaling activity\. When the scaling activity starts, you see an entry for the termination of the first instance and an entry for the launch of a new instance\. The **Instances** tab shows the new instance only\.  
![\[Auto Scaling group activity history\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-gs-group-activity-history.png)

1. On the navigation pane, choose **Instances**\. This page shows both the terminated instance and the running instance\.

Go to the next step if you would like to delete your basic infrastructure for automatic scaling\. Otherwise, you can use this infrastructure as your base and try one or more of the following:
+ [Manual Scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)
+ [Dynamic Scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)
+ [Getting Amazon SNS Notifications When Your Auto Scaling Group Scales](ASGettingNotifications.md)

## Step 4: \(Optional\) Delete Your Scaling Infrastructure<a name="gs-delete-asg"></a>

You can either delete your scaling infrastructure or delete just your Auto Scaling group and keep your launch template to use later\.

**To delete your Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. Choose **Actions**, **Delete**\. When prompted for confirmation, choose **Yes, Delete**\.

   The **Name** column indicates that the Auto Scaling group is being deleted\. The **Desired**, **Min**, and **Max** columns show `0` instances for the Auto Scaling group\.

Skip this procedure if you would like to keep your launch template\.

**To delete your launch template**

1. On the navigation pane, choose **Instances**, **Launch Templates**\.

1. Select your launch template\.

1. Choose **Actions**, **Delete template**\. When prompted for confirmation, choose **Delete launch template**\.

Skip this procedure if you would like to keep your launch configuration\.

**To delete your launch configuration**

1. On the navigation pane, under **Auto Scaling**, choose **Launch Configurations**\.

1. Select your launch configuration\.

1. Choose **Actions**, **Delete launch configuration**\. When prompted for confirmation, choose **Yes, Delete**\.