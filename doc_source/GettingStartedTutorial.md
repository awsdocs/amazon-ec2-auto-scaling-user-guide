# Getting Started with Amazon EC2 Auto Scaling<a name="GettingStartedTutorial"></a>

When you use Amazon EC2 Auto Scaling, you must use certain building blocks to get started\. This tutorial walks you through the process for setting up the basic infrastructure for Amazon EC2 Auto Scaling\.

The following step\-by\-step instructions help you create a template that defines your EC2 instances, create an Auto Scaling group to maintain the healthy number of instances at all times, and optionally delete this basic infrastructure\. This tutorial assumes that you are familiar with launching EC2 instances and have already created a key pair and a security group\.

**Topics**
+ [Step 1: Create a Launch Template](#gs-create-lt)
+ [Step 2: Create an Auto Scaling Group](#gs-create-asg)
+ [Step 3: Verify Your Auto Scaling Group](#gs-verify-asg)
+ [Step 4: \(Optional\) Delete Your Scaling Infrastructure](#gs-delete-asg)

## Step 1: Create a Launch Template<a name="gs-create-lt"></a>

A launch template specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. When you create a launch template, you include information such as the ID of the Amazon Machine Image \(AMI\) to use for launching the EC2 instance, the instance type, key pairs, security groups, and block device mappings, among other settings\.

**To create a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, select a region\. The Amazon EC2 Auto Scaling resources that you create are tied to the region you specify and are not replicated across regions\. For more information, see [Example: Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. On the navigation pane, choose **Instances**, **Launch Templates**\.

1. Choose **Create launch template**\.

1. Provide a name and description for the launch template\. The name of a launch template must have 3 to 125 characters, and can include the following characters: \(\)\./\_\.

1. For **AMI ID**, type the ID of the AMI for your instances\.

1. For **Instance type**, select a hardware configuration for your instances that is compatible with the AMI that you specified\.

1. \(Optional\) For **Key pair name**, type the name of the key pair to use when connecting to your instances\.

1. For **Network interfaces**, configure **Subnet**, **Auto\-assign public IP**, and **Security group ID**\. You can leave the other fields empty and this creates a primary network interface with IP addresses that we select for your instances\.

1. \(Optional\) For **Storage \(Volumes\)**, specify volumes to attach to the instances in addition to the volumes specified by the AMI you specified\.

1. \(Optional\) For **Tags**, specify one or more tags to associate with the instances and volumes\.

1. Choose **Create launch template**\.

1. On the confirmation page, choose **Create Auto Scaling group**\.

## Step 2: Create an Auto Scaling Group<a name="gs-create-asg"></a>

An Auto Scaling group is a collection of EC2 instances, and the core of Amazon EC2 Auto Scaling\. When you create an Auto Scaling group, you include information such as the launch template, the subnets for the instances, and the number of instances the group must maintain at all times\.

**To create an Auto Scaling group**

1. On the **Configure Auto Scaling group details** page, do the following:

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default or the latest version of the launch template when scaling out\.

   1. For **Group name**, type a name for your Auto Scaling group \(for example, `my-first-asg`\)\.

   1. Keep **Group size** set to the default value of `1` instance for this tutorial\.

   1. \(Optional\) To override the network in the launch template, select a virtual private cloud \(VPC\) for **Network**\.

   1. \(Optional\) To override the subnets in the launch template, select one or more subnets for **Subnet**\. 

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

1. \(Optional\) If you want, you can try the following experiment to learn more about Amazon EC2 Auto Scaling\. The minimum size for your Auto Scaling group is 1 instance\. Therefore, if you terminate the running instance, Amazon EC2 Auto Scaling must launch a new instance to replace it\.

   1. On the **Instances** tab, select the ID of the instance\. This shows you the instance on the **Instances** page\.

   1. Choose **Actions**, **Instance State**, **Terminate**\. When prompted for confirmation, choose **Yes, Terminate**\.

   1. On the navigation pane, choose **Auto Scaling Groups**\. Select your Auto Scaling group and choose the **Activity History** tab\. The default cooldown for the Auto Scaling group is 300 seconds \(5 minutes\), so it takes about 5 minutes until you see the scaling activity\. When the scaling activity starts, you'll see an entry for the termination of the first instance and an entry for the launch of a new instance\. The **Instances** tab shows the new instance only\.

   1. On the navigation pane, choose **Instances**\. This page shows both the terminated instance and the running instance\.

Go to the next step if you would like to delete your basic infrastructure for automatic scaling\. Otherwise, you can use this infrastructure as your base and try one or more of the following:
+ [Maintaining the Number of Instances in Your Auto Scaling Group](as-maintain-instance-levels.md)
+ [Manual Scaling](as-manual-scaling.md)
+ [Dynamic Scaling](as-scale-based-on-demand.md)
+ [Getting SNS Notifications When Your Auto Scaling Group Scales](ASGettingNotifications.md)

## Step 4: \(Optional\) Delete Your Scaling Infrastructure<a name="gs-delete-asg"></a>

You can either delete your scaling infrastructure or delete just your Auto Scaling group and keep your launch template to use at a later time\.

**To delete your Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group \(for example, **my\-first\-asg**\)\.

1. Choose **Actions**, **Delete**\. When prompted for confirmation, choose **Yes, Delete**\.

   The **Name** column indicates that the Auto Scaling group is being deleted\. The **Desired**, **Min**, and **Max** columns shows `0` instances for the Auto Scaling group\.

Skip this procedure if you would like keep your launch template\.

**To delete your launch template**

1. On the navigation pane, choose **Instances**, **Launch Templates**\.

1. Select your launch template \(for example, **my\-first\-lt**\)\.

1. Choose **Actions**, **Delete template**\. When prompted for confirmation, choose **Delete launch template**\.