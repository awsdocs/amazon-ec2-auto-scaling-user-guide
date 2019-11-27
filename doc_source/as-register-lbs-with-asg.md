# Tutorial: Set Up a Scaled and Load\-Balanced Application<a name="as-register-lbs-with-asg"></a>

You can attach a load balancer to your Auto Scaling group\. The load balancer automatically distributes incoming traffic across the instances in the group\. 

This tutorial attaches a load balancer to an Auto Scaling group when you create the group\. To attach a load balancer to an existing Auto Scaling group, see [Attaching a Load Balancer to Your Auto Scaling Group](attach-load-balancer-asg.md)\.

Before you explore this tutorial, we recommend that you first review the following introductory topic: [Using a Load Balancer with an Auto Scaling Group](autoscaling-load-balancer.md)\.

**Topics**
+ [Prerequisites](#as-register-lbs-prerequisites)
+ [Configure Scaling and Load Balancing \(Console\)](#as-register-lbs-console)
+ [Configure Scaling and Load Balancing \(AWS CLI\)](#as-lbs-app-cli)
+ [Cleaning Up Your AWS Resources](#as-lbs-app-clean-up)

## Prerequisites<a name="as-register-lbs-prerequisites"></a>
+ \(Optional\) Create an IAM role that grants your application the access to AWS that it needs\.
+ Launch an instance\. Be sure to specify the IAM role \(if you created one\), and specify any configuration scripts that you need as user data\. Connect to the instance and customize it\. For example, you can install software and applications and copy data\. Test your application on your instance to ensure that your instance is configured correctly\. Create a custom Amazon Machine Image \(AMI\) from your instance\. You can terminate the instance if you no longer need it\.
+ Create a load balancer\. Elastic Load Balancing supports three types of load balancers: Application Load Balancers, Network Load Balancers, and Classic Load Balancers\. You can attach any of these types of load balancers to your Auto Scaling group\. For more information, see [ Elastic Load Balancing Types](autoscaling-load-balancer.md#integrations-aws-elastic-load-balancing-types)\.
+ Make sure that you choose the same Availability Zones for the load balancer that you plan to enable for your Auto Scaling group\. 

## Configure Scaling and Load Balancing \(Console\)<a name="as-register-lbs-console"></a>

Complete the following tasks to set up a scaled and load\-balanced application when you create your Auto Scaling group\.

**Topics**
+ [Create or Select a Launch Configuration](#as-register-lbs-create-lc-console)
+ [Create or Select a Launch Template](#as-register-lbs-create-lt-console)
+ [Create an Auto Scaling Group](#as-register-lbs-create-asg-console)
+ [\(Optional\) Verify That Your Load Balancer Is Attached to Your Auto Scaling Group](#as-register-lbs-verify-console)

### Create or Select a Launch Configuration<a name="as-register-lbs-create-lc-console"></a>

A launch configuration specifies the type of EC2 instance that Amazon EC2 Auto Scaling creates for you\. When you create a launch configuration, you include information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, key pair, and block device mapping\. If you created a launch template, you can use your launch template to create an Auto Scaling group instead of using a launch configuration\. For more information, see [Create or Select a Launch Template](#as-register-lbs-create-lt-console)\.

If you already have a launch configuration that you'd like to use, select it by using the following procedure\.

**To select an existing launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the AWS Region that you used when creating your load balancer\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. On the next page, choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration**, select an existing launch configuration, and then choose **Next Step**\. 

To create a new launch configuration, use the following procedure:

**To create a launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the AWS Region that you used when creating your load balancer\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\. 

1. On the next page, choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration**, **Create a new launch configuration**, and then choose **Next Step**\. 

1. On the **Choose AMI** page, select your custom AMI\.

1. On the **Choose Instance Type** page, select a hardware configuration for your instance, and then choose **Next: Configure details**\.

1. On the **Configure Details** page, do the following:

   1. For **Name**, enter a name for your launch configuration\.

   1. \(Optional\) To securely distribute credentials to your EC2 instance, select your IAM role\.

   1. \(Optional\) If you need to connect to instances in a nondefault VPC, for **Advanced Details**, **IP Address Type**, choose **Assign a public IP address to every instance**\.

   1. \(Optional\) To specify user data or a configuration script for your instance, for **Advanced Details**, **User data**, paste your configuration script\.

   1. Choose **Skip to review**\.

1. On the **Review** page, choose **Edit security groups**\. Follow the instructions to choose an existing security group, and then choose **Review**\.

1. On the **Review** page, choose **Create launch configuration**\.

1. On the **Select an existing key pair or create a new key pair** page, select one of the listed options\. Select the acknowledgment check box, and then choose **Create launch configuration**\.
**Warning**  
Do not choose **Proceed without a key pair** if you need to connect to your instance\.

After completing the instructions above, you're ready to proceed with the wizard to create an Auto Scaling group\.

### Create or Select a Launch Template<a name="as-register-lbs-create-lt-console"></a>

If you already have a launch template that you'd like to use, select it using the following procedure\.

**To select an existing launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the AWS Region that you used when creating your load balancer\.

1. On the navigation pane, choose **Launch Templates**\.

1. Select a launch template\.

1. Choose **Actions**, **Create Auto Scaling group**\.

Alternatively, to create a new launch template, use the following procedure\.

**To create a launch template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the AWS Region that you used when creating your load balancer\.

1. On the navigation pane, choose **Launch Templates**\.

1. Choose **Create launch template**\.

1. Provide a name and description for the launch template\. The name of a launch template must have 3 to 125 characters, and can include the following characters: \(\)\./\_\.

1. For **AMI ID**, enter the ID of the AMI for your instances\.

1. For **Instance type**, select a hardware configuration for your instances that is compatible with the AMI that you specified\.

1. \(Optional\) For **Key pair name**, enter the name of the key pair to use when connecting to your instances\.

1. For **Network interfaces**, do the following:

   1. Choose **Add network interface**\.

   1. \(Optional\) To connect to instances in a nondefault VPC, for **Auto\-assign public IP**, choose **Enable**\.

   1. For **Security group ID**, specify a security group for your instances\.

   1. For **Delete on termination**, choose whether the network interface is deleted when the Auto Scaling group scales in and terminates the instance to which the network interface is attached\. 

1. \(Optional\) To securely distribute credentials to your instances, for **Advanced details**, **IAM instance profile**, enter the Amazon Resource Name \(ARN\) of your IAM role\.

1. \(Optional\) To specify user data or a configuration script for your instances, paste it into **Advanced details**, **User data**\.

1. Choose **Create launch template**\.

1. On the confirmation page, choose **Create Auto Scaling group**\.

### Create an Auto Scaling Group<a name="as-register-lbs-create-asg-console"></a>

Use the following procedure to continue where you left off after selecting or creating your launch configuration or template\.

**To create an Auto Scaling group**

1. On the **Configure Auto Scaling group details** page, do the following:

   1. For **Group name**, enter a name for your Auto Scaling group\.

   1. \[Launch template\] For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

   1. \[Launch template\] For **Fleet Composition**, choose **Adhere to the launch template** to use the EC2 instance type and purchase option that are specified in the launch template\. 

   1. For **Group size**, enter the initial number of instances for your Auto Scaling group\.

   1. If you selected an instance type for your launch configuration or template that requires a VPC, such as a T2 instance, you must select a VPC for **Network**\. Otherwise, if your account supports EC2\-Classic and you selected an instance type that doesn't require a VPC, you can select either `Launch into EC2-Classic` or a VPC\.

   1. If you selected a VPC in the previous step, select one or more subnets from **Subnet**\. If you selected EC2\-Classic, select one or more Availability Zones from **Availability Zone\(s\)**\.

   1. For **Advanced Details**, select **Receive traffic from Elastic Load Balancer\(s\)** and then do one of the following:
      + \[Classic Load Balancers\] Select your load balancer from **Load Balancers**\.
      + \[Application/Network Load Balancers\] Select your target group from **Target Groups**\.

   1. \(Optional\) To use Elastic Load Balancing health checks, choose **ELB** for **Advanced Details**, **Health Check Type**\.

   1. Choose **Next: Configure scaling policies**\.

1. On the **Configure scaling policies** page, select **Keep this group at its initial size**, and then choose **Review**\.

   To configure scaling policies for your Auto Scaling group, see [Configure Scaling Policies](as-scaling-target-tracking.md#policy-creating-scalingpolicies-console)\.

1. Review the details of your Auto Scaling group\. You can choose **Edit** to make changes\. When you are finished, choose **Create Auto Scaling group**\.

### \(Optional\) Verify That Your Load Balancer Is Attached to Your Auto Scaling Group<a name="as-register-lbs-verify-console"></a>

**To verify that your load balancer is attached to your Auto Scaling group**

1. Select your Auto Scaling group\.

1. On the **Details** tab, **Load Balancers** shows any attached load balancers and **Target Groups** shows any attached target groups\.

1. On the **Activity History** tab, the **Status** column shows you the status of your Auto Scaling instances\. While an instance is launching, its status is `In progress`\. The status changes to `Successful` after the instance is launched\.

1. On the **Instances** tab, the **Lifecycle** column shows the state of your Auto Scaling instances\. After an instance is ready to receive traffic, its state is `InService`\.

   The **Health Status** column shows the result of the health checks on your instances\.

## Configure Scaling and Load Balancing \(AWS CLI\)<a name="as-lbs-app-cli"></a>

Complete the following tasks to set up a scaled and load\-balanced application\.

**Topics**
+ [Create a Launch Configuration](#as-lbs-app-create-lc-cli)
+ [Create a Launch Template](#as-lbs-app-create-lt-cli)
+ [Create an Auto Scaling Group with a Load Balancer](#as-lbs-app-create-asg-cli)

### Create a Launch Configuration<a name="as-lbs-app-create-lc-cli"></a>

If you already have a launch configuration that you'd like to use, skip this step\.

**To create the launch configuration**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command\. To assign public IP addresses to instances in a nondefault VPC, use the `--associate-public-ip-address` option as shown\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-launch-config \
  --image-id ami-01e24be29428c15b2 --instance-type t2.micro --associate-public-ip-address \
  --security-groups sg-eb2af88e
```

### Create a Launch Template<a name="as-lbs-app-create-lt-cli"></a>

If you already have a launch template that you'd like to use, skip this step\.

**To create the launch template**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command and pass a JSON file that contains the information for creating the launch template\. To assign public IP addresses to instances in a nondefault VPC, add the network interface attribute and set it to `"NetworkInterfaces":{"AssociatePublicIpAddress":true}` as shown\. 

```
aws ec2 create-launch-template --launch-template-name my-launch-template --version-description my-version-description \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"AssociatePublicIpAddress":true,"Groups":["sg-903004f8"],"DeleteOnTermination":true}],"ImageId":"ami-01e24be29428c15b2","InstanceType":"t2.micro"}'
```

### Create an Auto Scaling Group with a Load Balancer<a name="as-lbs-app-create-asg-cli"></a>

You can attach an existing load balancer to an Auto Scaling group when you create the group\. You can use either a launch configuration or a launch template to automatically configure the instances that your Auto Scaling group launches\. 

**To create an Auto Scaling group with an attached Classic Load Balancer**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command with the `--load-balancer-names` option to create an Auto Scaling group with an attached Classic Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --load-balancer-names "my-load-balancer" \
  --max-size 5 --min-size 1 --desired-capacity 2
```

**To create an Auto Scaling group with an attached target group for an Application Load Balancer or Network Load Balancer**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command with the `--target-group-arns` option to create an Auto Scaling group with an attached target group\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template "LaunchTemplateName=my-launch-template,Version=1" \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --target-group-arns "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/1234567890123456" \
  --max-size 5 --min-size 1 --desired-capacity 2
```

## Cleaning Up Your AWS Resources<a name="as-lbs-app-clean-up"></a>

You've now successfully completed the tutorial\. To avoid ongoing charges for resources that you aren't using, you should clean up the resources that you created just for this tutorial\. For step\-by\-step instructions, see [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)\.