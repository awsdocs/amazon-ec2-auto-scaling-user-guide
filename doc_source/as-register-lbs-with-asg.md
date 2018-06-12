# Tutorial: Set Up a Scaled and Load\-Balanced Application<a name="as-register-lbs-with-asg"></a>

You can attach a load balancer to your Auto Scaling group\. The load balancer automatically distributes incoming traffic across the instances in the group\. For more information about the benefits of using Elastic Load Balancing with Auto Scaling, see [Using a Load Balancer With an Auto Scaling Group](autoscaling-load-balancer.md)\.

This tutorial attaches a load balancer to an Auto Scaling group when you create the group\. To attach a load balancer to an existing Auto Scaling group, see [Attaching a Load Balancer to Your Auto Scaling Group](attach-load-balancer-asg.md)\.

**Topics**
+ [Prerequisites](#as-register-lbs-prerequisites)
+ [Configure Scaling and Load Balancing Using the AWS Management Console](#as-register-lbs-console)
+ [Configure Scaling and Load Balancing Using the AWS CLI](#as-lbs-app-cli)

## Prerequisites<a name="as-register-lbs-prerequisites"></a>
+ \(Optional\) Create an IAM role that grants your application the access to AWS that it needs\.
+ Launch an instance; be sure to specify the IAM role \(if you created one\) and specify any configuration scripts that you need as user data\. Connect to the instance and customize it\. For example, you can install software and applications and copy data\. Test your application on your instance to ensure that your instance is configured correctly\. Create a custom Amazon Machine Image \(AMI\) from your instance\. You can terminate the instance if you no longer need it\.
+ Create a load balancer\. Elastic Load Balancing supports three types of load balancers: Application Load Balancers, Network Load Balancers, and Classic Load Balancers\. You can attach any of these types of load balancers to your Auto Scaling group\. For more information, see the [Elastic Load Balancing User Guide](http://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/)\.

  With Classic Load Balancers, instances are registered with the load balancer\. With Application Load Balancers and Network Load Balancers, instances are registered as targets with a target group\. When you plan to use your load balancer with an Auto Scaling group, you don't need to register your EC2 instances with the load balancer or target group\. After you attach a load balancer or target group to your Auto Scaling group, Auto Scaling registers your instances with the load balancer or target group when it launches them\.

## Configure Scaling and Load Balancing Using the AWS Management Console<a name="as-register-lbs-console"></a>

Complete the following tasks to set up a scaled and load\-balanced application when you create your Auto Scaling group\.

**Topics**
+ [Create or Select a Launch Configuration](#as-register-lbs-create-lc-console)
+ [Create an Auto Scaling Group](#as-register-lbs-create-asg-console)
+ [\(Optional\) Verify that Your Load Balancer is Attached to Your Auto Scaling Group](#as-register-lbs-verify-console)

### Create or Select a Launch Configuration<a name="as-register-lbs-create-lc-console"></a>

A launch configuration specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. When you create a launch configuration, you include information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, key pair, and block device mapping\. If you created a launch template, you can use your launch template to create an Auto Scaling group instead of using a launch configuration\. For more information, see [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)\.

If you already have a launch configuration that you'd like to use, select it using the following procedure\.

**To select an existing launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the region that you used when creating your load balancer\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. On the next page, choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration**, select an existing launch configuration, and then choose **Next Step**\. 

To create a new launch configuration, use the following procedure:

**To create a launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the region that you used when creating your load balancer\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\. 

1. On the next page, choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration, Create a new launch configuration**, and then choose **Next Step**\. 

1. On the **Choose AMI** page, select your custom AMI\.

1. On the **Choose Instance Type** page, select a hardware configuration for your instance, and then choose **Next: Configure details**\.

1. On the **Configure Details** page, do the following:

   1. For **Name**, type a name for your launch configuration\.

   1. \(Optional\) To securely distribute credentials to your EC2 instance, select your IAM role\.

   1. \(Optional\) If you need to connect to an instance in a nondefault VPC, for **Advanced Details**, **IP Address Type**, choose **Assign a public IP address to every instance**\.

   1. \(Optional\) To specify user data or a configuration script for your instance, for **Advanced Details**, **User data**, paste your configuration script\.

   1. Choose **Skip to review**\.

1. On the **Review** page, choose **Edit security groups**\. Follow the instructions to choose an existing security group, and then choose **Review**\.

1. On the **Review** page, choose **Create launch configuration**\.

1. On the **Select an existing key pair or create a new key pair** page, select one of the listed options\. Select the acknowledgment check box, and then choose **Create launch configuration**\.
**Warning**  
Do not choose **Proceed without a key pair** if you need to connect to your instance\.

After completing the instructions above, you're ready to proceed with the wizard to create an Auto Scaling group\.

### Create an Auto Scaling Group<a name="as-register-lbs-create-asg-console"></a>

Use the following procedure to continue where you left off after selecting or creating your launch configuration\.

**To create an Auto Scaling group**

1. On the **Configure Auto Scaling group details** page, do the following:

   1. For **Group name**, type a name for your Auto Scaling group\.

   1. For **Group size**, type the initial number of instances for your Auto Scaling group\.

   1. If you selected an instance type for your launch configuration that requires a VPC, such as a T2 instance, you must select a VPC for **Network**\. Otherwise, if your account supports EC2\-Classic and you selected an instance type that doesn't require a VPC, you can select either `Launch into EC2-Classic` or a VPC\.

   1. If you selected a VPC in the previous step, select one or more subnets from **Subnet**\. If you selected EC2\-Classic instead, select one or more Availability Zones from **Availability Zone\(s\)**\.

   1. For **Advanced Details**, select `Receive traffic from Elastic Load Balancer(s)` and then do one of the following:
      + \[Classic Load Balancers\] Select your load balancer from **Load Balancers**\.
      + \[Target groups\] Select your target group from **Target Groups**\.

   1. \(Optional\) To use Elastic Load Balancing health checks, choose **ELB** for **Advanced Details**, **Health Check Type**\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, select **Keep this group at its initial size**, and then choose **Review**\.

   If you want to configure scaling policies for your Auto Scaling group, see [Create an Auto Scaling Group with Target Tracking Scaling Policies](as-scaling-target-tracking.md#policy_creating)\.

1. Review the details of your Auto Scaling group\. You can choose **Edit** to make changes\. When you are finished, choose **Create Auto Scaling group**\.

### \(Optional\) Verify that Your Load Balancer is Attached to Your Auto Scaling Group<a name="as-register-lbs-verify-console"></a>

**To verify that your load balancer is attached to your Auto Scaling group**

1. Select your Auto Scaling group\.

1. On the **Details** tab, **Load Balancers** shows any attached load balancers and **Target Groups** shows any attached target groups\.

1. On the **Details** tab, **Load Balancers** shows any attached load balancers\.

1. On the **Activity History** tab, the **Status** column shows you the status of your Auto Scaling instances\. While an instance is launching, its status is `In progress`\. The status changes to `Successful` after the instance is launched\.

1. On the **Instances** tab, the **Lifecycle** column shows the state of your Auto Scaling instances\. After an instance is ready to receive traffic, its state is `InService`\.

   The **Health Status** column shows the result of the health checks on your instances\.

## Configure Scaling and Load Balancing Using the AWS CLI<a name="as-lbs-app-cli"></a>

Complete the following tasks to set up a scaled and load\-balanced application\.

**Topics**
+ [Create a Launch Configuration](#as-lbs-app-create-lc-cli)
+ [Create an Auto Scaling Group with a Load Balancer](#as-lbs-app-create-asg-cli)

### Create a Launch Configuration<a name="as-lbs-app-create-lc-cli"></a>

If you already have a launch configuration that you'd like to use, skip this step\.

**To create the launch configuration**  
Use the following [create\-launch\-configuration](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command:

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc \
--image-id ami-514ac838 --instance-type m1.small
```

### Create an Auto Scaling Group with a Load Balancer<a name="as-lbs-app-create-asg-cli"></a>

You can attach an existing load balancer to an Auto Scaling group when you create the group\.

**To create an Auto Scaling group with an attached Classic Load Balancer**  
Use the following [create\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command with the `--load-balancer-names` option to create an Auto Scaling group with an attached Classic Load Balancer:

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-lb-asg \
--launch-configuration-name my-lc \
--availability-zones "us-west-2a" "us-west-2b" \
--load-balancer-names "my-lb" \
--max-size 5 --min-size 1 --desired-capacity 2
```

**To create an Auto Scaling group with an attached target group**  
Use the following [create\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command with the `--target-group-arns` option to create an Auto Scaling group with an attached target group:

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-lb-asg \
--launch-configuration-name my-lc \
--vpc-zone-identifier "subnet-41767929" \ 
--vpc-zone-identifier "subnet-b7d581c0" \
--target-group-arns "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-targets/1234567890123456" \
--max-size 5 --min-size 1 --desired-capacity 2
```