# Creating a Launch Configuration<a name="create-launch-config"></a>

When you create a launch configuration, you must specify information about the EC2 instances to launch\. Include the Amazon Machine Image \(AMI\), instance type, key pair, security groups, and block device mapping\. Alternatively, you can create a launch configuration using attributes from a running EC2 instance\. For more information, see [Creating a Launch Configuration Using an EC2 Instance](create-lc-with-instanceID.md)\.

**Note**  
When you create an Auto Scaling group, you can specify a launch template, launch configuration, or an EC2 instance\. We recommend that you use a launch template to ensure that you can use the latest features of Amazon EC2\. For more information, see [Launch Templates](LaunchTemplates.md)\.

After you create a launch configuration, you can create an Auto Scaling group\. For more information, see [Creating an Auto Scaling Group Using a Launch Configuration](create-asg.md)\.

An Auto Scaling group is associated with one launch configuration at a time, and you can't modify a launch configuration after you've created it\. Therefore, if you want to change the launch configuration for an existing Auto Scaling group, you must update it with the new launch configuration\. For more information, see [Changing the Launch Configuration for an Auto Scaling Group](change-launch-config.md)\.

**To create a launch configuration using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, the current region is displayed\. Select a region for your Auto Scaling group that meets your needs\.

1. On the navigation pane, under **Auto Scaling**, choose **Launch Configurations**\. 

1. On the next page, choose **Create launch configuration**\.

1. On the **Choose AMI** page, select an AMI\.

1. On the **Choose Instance Type** page, select a hardware configuration for your instance\. Choose **Next: Configure details**\.
**Note**  
T2 instances must be launched into a subnet of a VPC\. If you select a `t2.micro` instance but don't have a VPC, one is created for you\. This VPC includes a public subnet in each Availability Zone in the region\.

1. On the **Configure Details** page, do the following:

   1. For **Name**, type a name for your launch configuration\.

   1. \(Optional\) For **Purchasing option**, you may request Spot Instances and specify the maximum price you are willing to pay per instance hour\. For more information, see [Launching Spot Instances in Your Auto Scaling Group](asg-launch-spot-instances.md)\.

   1. \(Optional\) For **IAM role**, select a role to associate with the instances\. For more information, see [IAM Role for Applications That Run on Amazon EC2 Instances](us-iam-role.md)\.

   1. \(Optional\) By default, basic monitoring is enabled for your Auto Scaling instances\. To enable detailed monitoring for your Auto Scaling instances, select **Enable CloudWatch detailed monitoring**\.

   1. For **Advanced Details**, **IP Address Type**, select an option\. To connect to instances in a VPC, you must select an option that assigns a public IP address\. If you want to connect to your instances but aren't sure whether you have a default VPC, select **Assign a public IP address to every instance**\.

   1. Choose **Skip to review**\.

1. On the **Review** page, choose **Edit security groups**\. Follow the instructions to choose an existing security group, and then choose **Review**\.

1. On the **Review** page, choose **Create launch configuration**\.

1. For **Select an existing key pair or create a new key pair**, select one of the listed options\. Select the acknowledgment check box, and then choose **Create launch configuration**\.
**Warning**  
If you need to connect to your instance, do not select **Proceed without a key pair**\.

**To create a launch configuration using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASLaunchConfiguration.html](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASLaunchConfiguration.html) \(AWS Tools for Windows PowerShell\)