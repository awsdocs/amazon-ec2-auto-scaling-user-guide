# Creating an Auto Scaling group using the Amazon EC2 launch wizard<a name="create-asg-ec2-wizard"></a>

You can create a launch configuration and an Auto Scaling group in a single procedure by using the Amazon EC2 launch wizard\. This is useful if you're launching more than one instance, and want to create a new launch configuration and Auto Scaling group from settings you've already selected in the Amazon EC2 launch wizard\. You cannot use this option to create an Auto Scaling group using an existing launch configuration\.

**To create a launch configuration and Auto Scaling group using the launch wizard**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. From the dashboard, choose **Launch Instance**\.

1. Choose an AMI, then choose an instance type on the next page, and then choose **Next: Configure Instance Details**\.

1. In **Number of instances**, enter the number of instances that you want to launch, and then choose **Launch into Auto Scaling Group**\. You do not need to add any other configuration details on the page\. 

1. On the confirmation page, choose **Create Launch Configuration**\. 

1. You are switched to step 3 of the launch configuration wizard\. The AMI and instance type are already selected based on the selection you made in the Amazon EC2 launch wizard\. Enter a name for the launch configuration, configure any other settings as required, and then choose **Next: Add Storage**\.

1. Configure any additional volumes, and then choose **Next: Configure Security Group**\.

1. Create a new security group, or choose an existing group, and then choose **Review**\.

1. Review the details of the launch configuration, and then choose **Create launch configuration** to choose a key pair and create the launch configuration\.

1. On the **Configure Auto Scaling group details** page, the launch configuration that you created is already selected for you, and the number of instances you specified in the Amazon EC2 launch wizard is populated for **Group size**\. Enter a name for the group, specify a VPC and subnet \(if required\), and then choose **Next: Configure scaling policies**\. 

1. On the **Configure scaling policies** page, choose one of the following options, and then choose **Review**:
   + To manually adjust the size of the Auto Scaling group as needed, choose **Keep this group at its initial size**\. For more information, see [Manual scaling for Amazon EC2 Auto Scaling](as-manual-scaling.md)\.
   + To automatically adjust the size of the Auto Scaling group based on criteria that you specify, choose **Use scaling policies to adjust the capacity of this group** and follow the directions\. For more information, see [Configure Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

1. On the **Review** page, you can optionally add tags or notifications, and edit other configuration details\. When you have finished, choose **Create Auto Scaling group**\.