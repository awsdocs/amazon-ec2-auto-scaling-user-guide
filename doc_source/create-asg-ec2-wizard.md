# Creating an Auto Scaling group using the Amazon EC2 launch wizard<a name="create-asg-ec2-wizard"></a>

You can create a launch template and an Auto Scaling group in a single procedure by using the Amazon EC2 launch wizard\. This is useful if you want to create a new launch template and Auto Scaling group from settings you've already selected in the Amazon EC2 launch wizard\. You cannot use this option to create an Auto Scaling group using an existing launch template\.

**To create a launch template and Auto Scaling group using the launch wizard**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. From the dashboard, choose **Launch instances**\.

1. Choose an AMI, then choose an instance type on the next page, and then choose **Next: Configure Instance Details**\.

1. In **Step 3: Configure Instance Details**, choose **Launch into Auto Scaling Group** next to **Number of instances**\.

1. On the confirmation page, choose **Continue**\.

1. On the **Create launch template** page, enter a name and description for the new launch template, and review the details of the template\. If everything is satisfactory, choose **Create launch template**\. Otherwise, update the settings of the launch template\. For more information, see [Creating your launch template \(console\)](create-launch-template.md#create-launch-template-for-auto-scaling)\.

1. On the confirmation page, choose **Create Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, the launch template that you created is already selected for you\. Enter a name for the group, and then choose **Next**\.

1. On the **Configure settings** page, for **Purchase options and instance types**, choose **Adhere to the launch template** to use the EC2 instance type and purchase option that are specified in the launch template\. 

1. Under **Network**, specify a VPC and one or more subnets\.

1. Choose **Next** twice\. 

1. \(Optional\) On the **Configure group size and scaling policies** page, configure the following options, and then choose **Next**:

   1. For **Desired capacity**, enter the initial number of instances to launch\. When you change this number to a value outside of the minimum or maximum capacity limits, you must update the values of **Minimum capacity** or **Maximum capacity**\. For more information, see [Setting capacity limits for your Auto Scaling group](asg-capacity-limits.md)\.

   1. To automatically scale the size of the Auto Scaling group, choose **Target tracking scaling policy** and follow the directions\. For more information, see [Target Tracking Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

1. You can optionally add notifications or tags\. Or, you can choose **Skip to review**\. 

1. On the **Review** page, choose **Create Auto Scaling group**\.