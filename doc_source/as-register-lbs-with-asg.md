# Tutorial: Set up a scaled and load\-balanced application<a name="as-register-lbs-with-asg"></a>

**Important**  
Before you explore this tutorial, we recommend that you first review the following introductory tutorial: [Getting started with Amazon EC2 Auto Scaling](GettingStartedTutorial.md)\.

Registering your Auto Scaling group with an Elastic Load Balancing load balancer helps you set up a load\-balanced application\. Elastic Load Balancing works with Amazon EC2 Auto Scaling to distribute incoming traffic across your healthy Amazon EC2 instances\. This increases the scalability and availability of your application\. You can enable Elastic Load Balancing within multiple Availability Zones to increase the fault tolerance of your applications\. 

In this tutorial, we cover the basics steps for setting up a load\-balanced application when the Auto Scaling group is created\. To register an existing Auto Scaling group with a load balancer, see [Attaching a load balancer](attach-load-balancer-asg.md) instead\.

Elastic Load Balancing supports different types of load balancers, including Application Load Balancers, Network Load Balancers, and Classic Load Balancers\. In most cases, we recommend that you use an Application Load Balancer or a Network Load Balancer\. However, you can still use a Classic Load Balancer if it supports the features that your application needs\. For more information, see [Elastic Load Balancing and Amazon EC2 Auto Scaling](autoscaling-load-balancer.md)\.

The procedure for deploying this scalable, load\-balanced architecture for dynamic traffic consists of the following steps\. 
+ \(Optional\) Create the launch template or launch configuration that will serve as a template for your instances\. Choose either the launch template or the launch configuration instructions, based on your preference\. Skip this step if you want to use your own launch template or launch configuration\.
+ Next, create your Auto Scaling group and attach the load balancer\. 
+ Finally, verify that your load balancer is attached\.

**Topics**
+ [Prerequisites](#as-register-lbs-prerequisites)
+ [Deploy your application \(console\)](#as-register-lbs-console)
+ [Deploy your application \(AWS CLI\)](#as-lbs-app-cli)
+ [Next steps](#as-lbs-app-next-steps)
+ [Clean up your AWS resources](#as-lbs-app-clean-up)

## Prerequisites<a name="as-register-lbs-prerequisites"></a>
+ You have a load balancer and target group created to use\. With Application Load Balancers or Network Load Balancers, your Auto Scaling group is registered to a target group that is associated with the load balancer\. Make sure that you choose the same Availability Zones for the load balancer that you plan to enable for your Auto Scaling group\. For more information, see [Getting started with Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-getting-started.html) in the *Elastic Load Balancing User Guide*\.
+ You have a security group for your launch template or launch configuration that allows access from the load balancer on the listener port \(usually port 80 for HTTP traffic\) and the port on which you want Elastic Load Balancing to perform health checks\. For more information, see the applicable documentation:
  + [Target security groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-register-targets.html#target-security-groups) in the *User Guide for Application Load Balancers*
  + [Target security groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#target-security-groups) in the *User Guide for Network Load Balancers*
  + [Security groups for instances in a VPC](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-groups.html#elb-vpc-instance-security-groups) in the *User Guide for Classic Load Balancers*

  If your instances will have public IP addresses, you can also optionally allow SSH traffic if you need to connect to the instances\. 
+ \(Optional\) You have an IAM role that grants your application the access to AWS that it needs\.
+ \(Optional\) You have an Amazon Machine Image \(AMI\) defined to be the source template for your Amazon EC2 instances\. To create one now, launch an instance\. Specify the IAM role \(if you created one\) and any configuration scripts that you need as user data\. Connect to the instance and customize it\. For example, installing software and applications, copying data, and attaching additional EBS volumes\. Test your applications on your instance to ensure that your instance is configured correctly\. Save this updated configuration as a custom AMI\. You can terminate the instance if you won't need it later\. Instances launched from this new custom AMI include the customizations that you made when you created the AMI\. 
+ This tutorial refers to the default VPC, but you can use your own VPC\. In the latter case, make sure that your VPC has a subnet mapped to each Availability Zone of the AWS Region you are working in\. At minimum, you must have two public subnets available to create the load balancer and either two private subnets or two public subnets to create your Auto Scaling group and register it with the load balancer\.

## Deploy your application \(console\)<a name="as-register-lbs-console"></a>

The following sections step you through the process of deploying your application\. 

**Topics**
+ [Create a launch template](#as-register-lbs-create-lt-console)
+ [Create a launch configuration](#as-register-lbs-create-lc-console)
+ [Create an Auto Scaling group](#as-register-lbs-create-asg-console)
+ [\(Optional\) Verify that your load balancer is attached](#as-register-lbs-verify-console)

### Create a launch template<a name="as-register-lbs-create-lt-console"></a>

You must have either a launch template or a launch configuration\. If you already have a launch template that you'd like to use, skip this step\.

**To create a launch template \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, choose the AWS Region where the load balancer was created\.

1. On the navigation pane, under **INSTANCES**, choose **Launch Templates**\.

1. Choose **Create launch template**\. 

1. Enter a name and provide a description for the initial version of the launch template\.

1. For **Amazon machine image \(AMI\)**, enter the ID of the AMI for your instances as search criteria\. 

1. For **Instance type**, select a hardware configuration for your instances that is compatible with the AMI that you specified\.

1. \(Optional\) For **Key pair \(login\)**, choose the key pair to use when connecting to your instances\.

1. For **Network interfaces**, do the following:

   1. Choose **Add network interface**\. We will use the option to specify the network interface for the purpose of this tutorial\.

   1. \(Optional\) For **Auto\-assign public IP**, keep the default value, **Don't include in launch template**\. When you create your Auto Scaling group, you can assign a public IP address to instances in your Auto Scaling group by using subnets that have the public IP addressing attribute enabled, such as the default subnets in the default VPC\. Alternatively, if you don't need to connect to your instances, you can choose **Disable** to prevent instances in your group from receiving traffic directly from the Internet\. In this case, they will receive traffic only from the load balancer\.

   1. For **Security group ID**, specify a security group for your instances from the same VPC as the load balancer\. 

   1. For **Delete on termination**, choose **Yes** so that the network interface is deleted when the Auto Scaling group scales in and terminates the instance to which the network interface is attached\. 

1. \(Optional\) To securely distribute credentials to your instances, for **Advanced details**, **IAM instance profile**, enter the Amazon Resource Name \(ARN\) of your IAM role\.

1. \(Optional\) To specify user data or a configuration script for your instances, paste it into **Advanced details**, **User data**\.

1. Choose **Create launch template**\. 

1. On the confirmation page, choose **Create Auto Scaling group**\.

### Create a launch configuration<a name="as-register-lbs-create-lc-console"></a>

If you already have a launch configuration that you'd like to use, skip this step\.

**To create a launch configuration \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\. 

1. On the navigation bar at the top of the screen, choose the AWS Region where the load balancer was created\.

1. Choose **Create launch configuration**, and enter a name for your launch configuration\. 

1. For **Amazon machine image \(AMI\)**, enter the ID of the AMI for your instances as search criteria\. 

1. For **Instance type**, select a hardware configuration for your instance\.

1. Under **Additional configuration**, pay attention to the following fields:

   1. \(Optional\) To securely distribute credentials to your EC2 instance, for **IAM instance profile**, select your IAM role\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

   1. \(Optional\) To specify user data or a configuration script for your instance, paste it into **Advanced details**, **User data**\.

   1. \(Optional\) For **Advanced details**, **IP address type**, keep the default value\. When you create your Auto Scaling group, you can assign a public IP address to instances in your Auto Scaling group by using subnets that have the public IP addressing attribute enabled, such as the default subnets in the default VPC\. Alternatively, if you don't need to connect to your instances, you can choose **Do not assign a public IP address to any instances** to prevent instances in your group from receiving traffic directly from the Internet\. In this case, they will receive traffic only from the load balancer\.

1. For **Security groups**, choose an existing security group from the same VPC as the load balancer\. If you leave the **Create a new security group** option selected, a default SSH rule is configured for Amazon EC2 instances running Linux\. A default RDP rule is configured for Amazon EC2 instances running Windows\. 

1. For **Key pair \(login\)**, choose an option under **Key pair options**\. 

   If you've already configured an Amazon EC2 instance key pair, you can choose it here\. 

   If you don't already have an Amazon EC2 instance key pair, choose **Create a new key pair** and give it a recognizable name\. Choose **Download key pair** to download the key pair to your computer\. 
**Important**  
If you need to connect to your instances, do not choose **Proceed without a key pair**\.

1. Select the acknowledgment check box, and then choose **Create launch configuration**\.

1. Select the check box next to the name of your new launch configuration and choose **Actions**, **Create Auto Scaling group**\. 

### Create an Auto Scaling group<a name="as-register-lbs-create-asg-console"></a>

After completing the instructions above, you're ready to proceed with the wizard to create an Auto Scaling group\. 

Use the following procedure to continue where you left off after creating your launch template or launch configuration\. 

**To create an Auto Scaling group \(console\)**

1. If you have not yet navigated to the **Auto Scaling groups** page, do the following:

   1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

   1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group\.

1. For **Launch template**, choose one of the following options:

   1. \[Launch template only\] Choose the launch template that you created and then choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

   1. \[Launch configuration only\] Choose **Switch to launch configuration** and choose the launch configuration that you created\.

1. Choose **Next**\. 

   The **Configure settings** page appears, allowing you to configure network settings and giving you options for enabling diversification across On\-Demand Instances and Spot Instances of multiple instance types \(if you chose a launch template\)\. 

1. \[Launch template only\] Keep **Purchase options and instance types** set to **Adhere to the launch template** to use the EC2 instance type and purchase option that are specified in the launch template\. 

1. For **Network**, choose the VPC that you used for your load balancer\. If you chose the default VPC, it is automatically configured to provide internet connectivity to your instances\. This VPC includes a public subnet in each Availability Zone in the Region\. 

1. For **Subnet**, choose one or more subnets from each Availability Zone that you want to include, based on which Availability Zones the load balancer is in\.

1. Choose **Next**\. 

1. On the **Configure advanced options** page, under **Load balancing**, choose **Enable load balancing**\.

1. Do one of the following:

   1. Choose **Load balancer target groups**, and then choose the target group for the load balancer you created\.

   1. Choose **Classic Load Balancers**, and then choose your load balancer\.

1. \(Optional\) To use Elastic Load Balancing health checks, for **Health checks**, choose **ELB** under **Health check type**\.

1. When you have finished configuring the Auto Scaling group, choose **Skip to review**\. 

1. On the **Review** page, review the details of your Auto Scaling group\. You can choose **Edit** to make changes\. When you are finished, choose **Create Auto Scaling group**\.

After you have created the Auto Scaling group with the load balancer attached, the load balancer automatically registers new instances as they come online\. You have only one instance at this point, so there isn't much to register\. However, you can now add additional instances by updating the desired capacity of the group\. For step\-by\-step instructions, see [Manual scaling](as-manual-scaling.md)\. 

### \(Optional\) Verify that your load balancer is attached<a name="as-register-lbs-verify-console"></a>

**To verify that your load balancer is attached \(console\)**

1. From the **Auto Scaling groups** page, select the check box next to your Auto Scaling group\.

1. On the **Details** tab, **Load balancing** shows any attached load balancer target groups or Classic Load Balancers\.

1. On the **Instance management** tab, under **Instances**, you can verify that your instances launched successfully\. Initially, your instances are in the `Pending` state\. After an instance is ready to receive traffic, its state is `InService`\. The **Health status** column shows the result of the Amazon EC2 Auto Scaling health checks on your instances\. Although an instance may be marked as healthy, the load balancer will only send traffic to instances that pass the load balancer health checks\.

1. Verify that your instances are registered with the load balancer\. On the navigation pane of the EC2 console, under **LOAD BALANCING**, choose **Target Groups**\. Select your target group, and then choose the **Targets** tab\. If the state of your instances is `initial`, it's probably because they are still in the process of being registered, or they have not passed the minimum number of health checks to be considered healthy\. When the state of your instances is `healthy`, they are ready for use\.

## Deploy your application \(AWS CLI\)<a name="as-lbs-app-cli"></a>

The following sections step you through the process of deploying your application\. 

**Topics**
+ [Create a launch template](#as-lbs-app-create-lt-cli)
+ [Create a launch configuration](#as-lbs-app-create-lc-cli)
+ [Create an Auto Scaling group with a load balancer](#as-lbs-app-create-asg-cli)

### Create a launch template<a name="as-lbs-app-create-lt-cli"></a>

If you already have a launch template that you'd like to use, skip this step\.

**To create the launch template**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command and pass a JSON file that contains the information for creating the launch template\.

```
aws ec2 create-launch-template --launch-template-name my-launch-template --version-description my-version-description \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"Groups":["sg-903004f8"],"DeleteOnTermination":true}],"ImageId":"ami-01e24be29428c15b2","InstanceType":"t2.micro"}'
```

### Create a launch configuration<a name="as-lbs-app-create-lc-cli"></a>

If you already have a launch configuration that you'd like to use, skip this step\.

**To create the launch configuration**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command\. 

```
aws autoscaling create-launch-configuration --launch-configuration-name my-launch-config \
  --image-id ami-01e24be29428c15b2 --instance-type t2.micro --security-groups sg-903004f8
```

### Create an Auto Scaling group with a load balancer<a name="as-lbs-app-create-asg-cli"></a>

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

## Next steps<a name="as-lbs-app-next-steps"></a>

Now that you have completed this tutorial, you can learn more:
+ You can configure your Auto Scaling group to use a scaling policy to automatically increase or decrease the number of instances as the demand on your instances changes\. This allows the group to handle changes in the amount of traffic that your application receives\. For more information, see [Target tracking scaling policies](as-scaling-target-tracking.md)\. 
+ If you would like to learn how to create automated schedules for scaling, [Scheduled scaling](schedule_time.md) provides an introduction to scheduling an Auto Scaling group to scale using one\-time or recurring scaling actions\.
+ You can configure your Auto Scaling group to use Elastic Load Balancing health checks\. If you enable load balancer health checks and an instance fails the health checks, the Auto Scaling group considers the instance unhealthy and replaces it\. For more information, see [Adding ELB health checks](as-add-elb-healthcheck.md)\.
+ You can expand your application to an additional Availability Zone in the same AWS Region to increase fault tolerance in the event of a service disruption\. For more information, see [Adding an Availability Zone](as-add-availability-zone.md)\.

## Clean up your AWS resources<a name="as-lbs-app-clean-up"></a>

You've now successfully completed the tutorial\. To avoid unnecessary charges to your account for resources that you aren't using, you should clean up the resources that you created just for this tutorial\. For step\-by\-step instructions, see [Deleting your Auto Scaling infrastructure](as-process-shutdown.md)\.