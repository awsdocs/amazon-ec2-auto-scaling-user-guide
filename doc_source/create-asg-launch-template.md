# Creating an Auto Scaling Group Using a Launch Template<a name="create-asg-launch-template"></a>

When you create an Auto Scaling group, you must specify the necessary information to configure the Amazon EC2 instances, the subnets for the instances, and the initial number of instances\. 

To configure Amazon EC2 instances, you can specify a launch configuration, a launch template, or an EC2 instance\. The following procedure demonstrates how to create an Auto Scaling group using a launch template\. 

With launch templates, you select the launch template and which specific version of the launch template the group uses to launch EC2 instances\. You can change these selections anytime by updating the group\. Alternatively, you can configure the Auto Scaling group to select either the default version or the latest version of the launch template dynamically when a scale\-out event occurs\. For example, you configure your Auto Scaling group to select the default version of a launch template dynamically\. To change the configuration of the EC2 instances to be launched by the group, create or designate a new default version of the launch template\. For more information, see [Launch Templates](LaunchTemplates.md)\.

**Prerequisites**
+ You must have an existing launch template that includes the parameters required to launch an EC2 instance\. For information about these parameters and the limitations that apply when creating a launch template for use with an Auto Scaling group, see [Creating a Launch Template for an Auto Scaling Group](create-launch-template.md)\.
+ You must have IAM permissions to create an Auto Scaling group using a launch template and also to create EC2 resources for the instances\. For more information, see [Controlling Access to Your Amazon EC2 Auto Scaling Resources](control-access-using-iam.md)\.

**To create an Auto Scaling group using a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the same region that you used when you created the launch template\.

1. In the navigation pane, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1. Choose **Launch Template**, select your launch template, and then choose **Next Step**\.

1. On the **Configure Auto Scaling group details** page, for **Group name**, type a name for your Auto Scaling group\.

1. For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

1. For **Fleet Composition**, choose **Adhere to the launch template** to use the EC2 instance type and purchase option specified in the launch template\. Choose **Combine purchase options and instances** to launch instances across multiple instance types using both On\-Demand and Spot purchase options\.

1. If you chose to combine purchase options and instance types:

   1. For **Instance Types**, choose the optimal instance types \(such as c5\.large\) that may be launched\. The order in which you add instance types sets their priority for On\-Demand Instances\. The instance type at the top of the list is prioritized the highest when the Auto Scaling group launches your On\-Demand capacity\. You must specify at least two instance types, up to a maximum of 20\.

   1. For **Instances Distribution**, choose whether to keep or replace the default instance distribution settings\. 

   1. If you chose to replace the default settings, provide the following information\. For more information about the options in this section, see [Using Multiple Instance Types and Purchase Options](AutoScalingGroup.md#asg-purchase-options)\.
      + For **Maximum Spot Price**, choose **Use default** to cap your maximum Spot price at the On\-Demand price or **Set your maximum price** to specify the maximum price you are willing to pay per instance per hour for Spot Instances\.
      + For **Spot Allocation Strategy**, choose the number of Spot pools to allocate your Spot Instances across\. 
      + For **Optional On\-Demand Base**, specify the minimum amount of the Auto Scaling group's initial capacity that must be fulfilled by On\-Demand Instances\. Leave this field blank to launch On\-Demand Instances as a percentage of the group's desired capacity\. 
      + For **On\-Demand Percentage Above Base**, specify the percentages of On\-Demand Instances and Spot Instances for your additional capacity beyond the optional On\-Demand base amount\. 

1. For **Group size**, enter the initial number of instances for your Auto Scaling group\.

1. For **Network**, choose a VPC for your Auto Scaling group\.
**Note**  
Launching instances using a combination of instance types and On\-Demand and Spot purchase options is not supported in EC2\-Classic\. 

1. For **Subnet**, choose one or more subnets in the specified VPC\. Use subnets in multiple Availability Zones for high availability\. For more information about high availability with Amazon EC2 Auto Scaling, see [Distributing Instances Across Availability Zones](auto-scaling-benefits.md#arch-AutoScalingMultiAZ)\.

1. \(Optional\) To register your Amazon EC2 instances with a load balancer, select **Receive traffic from one or more load balancers** and select one or more Classic Load Balancers or target groups\.

1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, select one of the following options, and then choose **Next: Configure Notifications**:
   + To manually adjust the size of the Auto Scaling group as needed, select **Keep this group at its initial size**\. For more information, see [Manual Scaling](as-manual-scaling.md)\.
   + To automatically adjust the size of the Auto Scaling group based on criteria that you specify, select **Use scaling policies to adjust the capacity of this group** and follow the directions\. For more information, see [Configure Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

1. \(Optional\) To receive notifications, choose **Add notification**, configure the notification, and then choose **Next: Configure Tags**\.

1. \(Optional\) To add tags, choose **Edit tags**, provide a tag key and value for each tag, and then choose **Review**\.

   Alternatively, you can add tags later on\. For more information, see [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)\.

1. On the **Review** page, choose **Create Auto Scaling group**\.

1. On the **Auto Scaling group creation status** page, choose **Close**\.

**To create an Auto Scaling group using the command line**

You can use one of the following commands:
+ [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)
+ [https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html](https://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)