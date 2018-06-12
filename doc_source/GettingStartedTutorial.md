# Getting Started with Amazon EC2 Auto Scaling<a name="GettingStartedTutorial"></a>

When you use Amazon EC2 Auto Scaling, you must use certain building blocks to get started\. This tutorial walks you through the process for setting up the basic infrastructure for Amazon EC2 Auto Scaling\.

The following step\-by\-step instructions help you create a configuration that defines your EC2 instances, create an Auto Scaling group to maintain the healthy number of instances at all times, and optionally delete this basic infrastructure\. This tutorial assumes that you are familiar with launching EC2 instances and have already created a key pair and a security group\.

If you created a launch template, you can use your launch template to create an Auto Scaling group instead of creating a launch configuration\. For more information, see [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

**Topics**
+ [Step 1: Create a Launch Configuration](#gs-create-lc)
+ [Step 2: Create an Auto Scaling Group](#gs-create-asg)
+ [Step 3: Verify Your Auto Scaling Group](#gs-verify-asg)
+ [Step 4: \(Optional\) Delete Your Scaling Infrastructure](#gs-delete-asg)

## Step 1: Create a Launch Configuration<a name="gs-create-lc"></a>

A launch configuration specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. You create the launch configuration by including information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, the key pair, security groups, and block device mapping\.

**To create a launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, select a region\. The Auto Scaling resources that you create are tied to the region you specify and are not replicated across regions\. For more information, see [Example: Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. On the **Welcome to Auto Scaling** page, choose **Create Auto Scaling group**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-console-first-time-user-screen.png)

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration, Create a new launch configuration**, and then choose **Next Step**\.

1. For the **Choose AMI** step, there is a list of basic configurations, called Amazon Machine Images \(AMIs\), that serve as templates for your instance\. Choose **Select** for the Amazon Linux AMI\.

1. For the **Choose Instance Type** step, select a hardware configuration for your instance\. We recommend that you keep the default, a `t2.micro` instance\. Choose **Next: Configure details**\.
**Note**  
T2 instances must be launched into a subnet of a VPC\. If you select a `t2.micro` instance but don't have a VPC, one is created for you\. This VPC includes a public subnet in each Availability Zone in the region\.

1. For the **Configure details** step, do the following:

   1. For **Name**, type a name for your launch configuration \(for example, `my-first-lc`\)\.

   1. For **Advanced Details**, select an IP address type\. If you want to connect to an instance in a VPC, you must select an option that assigns a public IP address\. If you want to connect to your instance but aren't sure whether you have a default VPC, select **Assign a public IP address to every instance**\.

   1. Choose **Skip to review**\.

1. For the **Review** step, choose **Edit security groups**\. Follow the instructions to choose an existing security group, and then choose **Review**\.

1. For the **Review** step, choose **Create launch configuration**\.

1. Complete the **Select an existing key pair or create a new key pair** step as instructed\. Note that you won't connect to your instance as part of this tutorial\. Therefore, you can select **Proceed without a key pair** unless you intend to connect to your instance\.

1. Choose **Create launch configuration**\. The launch configuration is created and the wizard to create an Auto Scaling group is displayed\.

## Step 2: Create an Auto Scaling Group<a name="gs-create-asg"></a>

An Auto Scaling group is a collection of EC2 instances, and the core of Amazon EC2 Auto Scaling\. When you create an Auto Scaling group, you include information such as the subnets for the instances and the number of instances the group must maintain at all times\.

Use the following procedure to continue where you left off after creating the launch configuration\.

**To create an Auto Scaling group**

1. For the **Configure Auto Scaling group details** step, do the following:

   1. For **Group name**, type a name for your Auto Scaling group \(for example, `my-first-asg`\)\.

   1. Keep **Group size** set to the default value of `1` instance for this tutorial\.

   1. Keep **Network** set to the default VPC for the region, or select your own VPC\.

   1. For **Subnet**, select one or more subnets for your Auto Scaling instances\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, select **Keep this group at its initial size** and choose **Review**\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

## Step 3: Verify Your Auto Scaling Group<a name="gs-verify-asg"></a>

Now that you have created your Auto Scaling group, you are ready to verify that the group has launched an EC2 instance\.

**To verify that your Auto Scaling group has launched an EC2 instance**

1. On the **Auto Scaling Groups** page, select the Auto Scaling group that you just created\.

1. On the **Activity History** tab, the **Status** column shows the current status of your instance\. While your instance is launching, the status column shows `In progress`\. The status changes to `Successful` after the instance is launched\. You can also use the refresh button to see the current status of your instance\.

1. On the **Instances** tab, the **Lifecycle** column shows the state of your instance\. You can see that your Auto Scaling group has launched your EC2 instance, and that it is in the `InService` lifecycle state\. The **Health Status** column shows the result of the EC2 instance health check on your instance\.

1. \(Optional\) If you want, you can try the following experiment to learn more about Amazon EC2 Auto Scaling\. The minimum size for your Auto Scaling group is 1 instance\. Therefore, if you terminate the running instance, Amazon EC2 Auto Scaling must launch a new instance to replace it\.

   1. On the **Instances** tab, select the ID of the instance\. This shows you the instance on the **Instances** page\.

   1. Choose **Actions**, **Instance State**, **Terminate**\. When prompted for confirmation, choose **Yes, Terminate**\.

   1. On the navigation pane, choose **Auto Scaling Groups**\. Select your Auto Scaling group and choose the **Activity History** tab\. The default cooldown for the Auto Scaling group is 300 seconds \(5 minutes\), so it takes about 5 minutes until you see the scaling activity\. When the scaling activity starts, you'll see an entry for the termination of the first instance and an entry for the launch of a new instance\. The **Instances** tab shows the new instance only\.

   1. On the navigation pane, choose **Instances**\. This page shows both the terminated instance and the running instance\.

Go to the next step if you would like to delete your basic infrastructure for automatic scaling\. Otherwise, you can use this infrastructure as your base and try one or more of the following:
+ [Maintaining the Number of Instances in Your Auto Scaling Group](as-maintain-instance-levels.md)
+ [Manual Scaling](as-manual-scaling.md)
+ [Dynamic Scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)
+ [Getting SNS Notifications When Your Auto Scaling Group Scales](ASGettingNotifications.md)

## Step 4: \(Optional\) Delete Your Scaling Infrastructure<a name="gs-delete-asg"></a>

You can either delete your scaling infrastructure or delete just your Auto Scaling group and keep your launch template to use at a later time\.

**To delete your Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group \(for example, **my\-first\-asg**\)\.

1. Choose **Actions**, **Delete**\. When prompted for confirmation, choose **Yes, Delete**\.

   The **Name** column indicates that the Auto Scaling group is being deleted\. The **Desired**, **Min**, and **Max** columns shows `0` instances for the Auto Scaling group\.

Skip this procedure if you would like keep your launch configuration\.

**To delete your launch configuration**

1. On the navigation pane, under **Auto Scaling**, choose **Launch Configurations**\.

1. Select your launch configuration \(for example, `my-first-lc`\)\.

1. Choose **Actions**, **Delete launch configuration**\. When prompted for confirmation, choose **Yes, Delete**\.