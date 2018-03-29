# Creating an Auto Scaling Group Using a Launch Template<a name="create-asg-launch-template"></a>

When you create an Auto Scaling group, you must specify the information needed to configure the Auto Scaling instances and the minimum number of instances your group must maintain at all times\.

To configure Auto Scaling instances, you must specify a launch template, a launch configuration, or an EC2 instance\. We recommend that you use a launch template to ensure that you can use the latest features of Amazon EC2\.

When you create an Auto Scaling group using a launch template, you specify which version of the launch template the Auto Scaling group uses to launch Auto Scaling instances\. You can specify a specific version \(using the API, AWS CLI, or an SDK, but not the console\)\. Alternatively, you can configure the Auto Scaling group to select either the default version or the latest version of the launch template dynamically when a scale out event occurs\.

If you configured your Auto Scaling group to select either the default version or the latest version of a launch template dynamically, you can change the configuration of the Auto Scaling instances to be launched by the group by creating a new version or new default version of the launch template\.

With the API, AWS CLI, or an SDK, when you update your Auto Scaling group, you can specify a different launch template for the group\. You can also specify a launch template when you update an Auto Scaling group that was created using a launch configuration\.

The following procedure demonstrates how to create an Auto Scaling group using a launch template\.

**Prerequisites**
+ Create a launch template\. You must ensure that your template includes all parameters required to launch an EC2 instance, such as an AMI ID and an instance type\. Otherwise, when you use the template to create an Auto Scaling group, you receive an error that you must use a fully\-formed launch template\. For more information, see [Launching an Instance from a Launch Template](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html) in the *Amazon EC2 User Guide for Linux Instances*\.
+ An IAM user or role that creates an Auto Scaling group using a launch template must have permission to use the `ec2:RunInstances` action and permission to create or use the resources for the instance\. For example, access to the `iam:PassRole` action is required to use an instance profile\. You can use the **AmazonEC2FullAccess** policy to grant full access to all Amazon EC2 resources\. You can use resource\-level permissions to restrict access to specific launch templates\. For more information, see [Require a Launch Template](control-access-using-iam.md#policy-example-launch-template) or [Launch Templates](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html#iam-example-runinstances-launch-templates) in the *Amazon EC2 User Guide for Linux Instances*\.

**Limitations**

The following are limitations when creating a launch template for use with an Auto Scaling group:
+ You cannot specify multiple network interfaces\.
+ If you specify a network interface, its device index must be 0\.
+ If you specify a network interface, you must specify any security groups as part of the network interface\.
+ You cannot specify private IP addresses\.
+ You cannot use host placement affinity\.
+ If you specify Spot Instances, you must specify a one\-time request with no end date\.

**To create an Auto Scaling group using a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the same region that you used when you created the launch template\.

1. In the navigation pane, choose **Launch Templates**\.

1. Select the launch template and choose **Actions**, **Create Auto Scaling group**\.

1. On the **Configure Auto Scaling group details** page, do the following:

   1. For **Launch template version**, choose whether the Auto Scaling group uses the default or the latest version of the launch template when scaling out\.

   1. For **Group name**, type a name for your Auto Scaling group\.

   1. For **Group size**, type the initial number of instances for your Auto Scaling group\.

   1. \(Optional\) To override the network in the launch template, select a VPC for **Network**\.

   1. \(Optional\) To override the network in the launch template, select one or more subnets for **Subnet**\.

   1. \(Optional\) To register your Auto Scaling instances with a load balancer, select **Receive traffic from one or more load balancers** and select one or more Classic Load Balancers or target groups\.

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
+ [create\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) \(AWS CLI\)
+ [New\-ASAutoScalingGroup](http://docs.aws.amazon.com/powershell/latest/reference/items/New-ASAutoScalingGroup.html) \(AWS Tools for Windows PowerShell\)