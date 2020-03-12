# Tutorial: Set Up a Scaled and Load\-Balanced Application<a name="as-register-lbs-with-asg"></a>

**Important**  
Before you explore this tutorial, we recommend that you first review the following introductory tutorial: [Getting Started with Amazon EC2 Auto Scaling](GettingStartedTutorial.md)\.

Registering your Auto Scaling group with an Elastic Load Balancing load balancer helps you set up a load\-balanced application\. Elastic Load Balancing works with Amazon EC2 Auto Scaling to load balance incoming traffic across your healthy Amazon EC2 instances\. This increases the scalability and availability of your application\. You can enable Elastic Load Balancing within a single Availability Zone or multiple Availability Zones to increase the fault tolerance of your applications\. For more information, see [Using a Load Balancer with an Auto Scaling Group](autoscaling-load-balancer.md)\. 

In this tutorial, we cover the basics steps for setting up a load\-balanced application when the Auto Scaling group is created\. To register an existing Auto Scaling group with a load balancer, see [Attaching a Load Balancer](attach-load-balancer-asg.md) instead\.

Elastic Load Balancing supports three types of load balancers: Application Load Balancers, Network Load Balancers, and Classic Load Balancers\. We recommend that you use either Application Load Balancers or Network Load Balancers\. However, you can still use a Classic Load Balancer if it supports the features that your application needs\. For more information, see [ Elastic Load Balancing Types](autoscaling-load-balancer.md#integrations-aws-elastic-load-balancing-types)\.

The procedure for deploying this scalable, load\-balanced architecture for dynamic traffic consists of the following steps\. 
+ First, choose either the launch template or the launch configuration instructions, based on your preference\. When you create a launch template or launch configuration, you include information such as the ID of the Amazon Machine Image \(AMI\) to use, the instance type, key pair, and block device mapping\. The required inputs are the instance type and AMI\. 
+ Next, configure your Auto Scaling group and attach the load balancer\. 
+ Finally, create your Auto Scaling group and verify that your load balancer is attached\.

**Topics**
+ [Prerequisites](#as-register-lbs-prerequisites)
+ [Deploy Your Application \(Console\)](#as-register-lbs-console)
+ [Deploy Your Application \(AWS CLI\)](#as-lbs-app-cli)
+ [Next Steps](#as-lbs-app-next-steps)
+ [Clean Up Your AWS Resources](#as-lbs-app-clean-up)

## Prerequisites<a name="as-register-lbs-prerequisites"></a>
+ You have an existing VPC and security group to use\. If you choose to use the default VPC, after you launch the load balancer you must ensure that its security group has appropriate inbound rules\. The default rule does not accept any inbound traffic\.
+ You have a load balancer created to use\. Make sure that you choose the same Availability Zones for the load balancer that you plan to enable for your Auto Scaling group\. 
+ \(Optional\) You have an IAM role that grants your application the access to AWS that it needs\.
+ \(Optional\) You have an Amazon Machine Image \(AMI\) defined to be the source template for your Amazon EC2 instances\. To create one now, launch an instance\. Specify the IAM role \(if you created one\) and any configuration scripts that you need as user data\. Connect to the instance and customize it\. For example, you can install software and applications on it\. Test your applications on your instance to ensure that your instance is configured correctly\. Save this updated configuration as a custom AMI\. You can terminate the instance if you won't need it later\. Instances launched from this new custom AMI include the customizations that you made when you created the AMI\. 

## Deploy Your Application \(Console\)<a name="as-register-lbs-console"></a>

The following sections step you through the process of deploying your application\. 

**Topics**
+ [Create or Select a Launch Template](#as-register-lbs-create-lt-console)
+ [Create or Select a Launch Configuration](#as-register-lbs-create-lc-console)
+ [Create an Auto Scaling Group](#as-register-lbs-create-asg-console)
+ [\(Optional\) Verify That Your Load Balancer Is Attached to Your Auto Scaling Group](#as-register-lbs-verify-console)

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

### Create or Select a Launch Configuration<a name="as-register-lbs-create-lc-console"></a>

If you already have a launch configuration that you'd like to use, select it by using the following procedure\.

**To select an existing launch configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, select the AWS Region that you used when creating your load balancer\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. On the next page, choose **Create Auto Scaling group**\.

1. On the **Create Auto Scaling Group** page, choose **Launch Configuration**, select an existing launch configuration, and then choose **Next Step**\. 

Alternatively, to create a new launch configuration, use the following procedure:

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
Do not choose **Proceed without a key pair** if you will need to connect to your instance\.

After completing the instructions above, you're ready to proceed with the wizard to create an Auto Scaling group\.

### Create an Auto Scaling Group<a name="as-register-lbs-create-asg-console"></a>

Use the following procedure to continue where you left off after selecting or creating your launch template or launch configuration\. 

**Note**  
Amazon EC2 Auto Scaling has changed the Auto Scaling group interface\. By default, you're shown the old user interface, but you can switch to the new user interface\. This section contains steps for both\.   
To switch to the new user interface, on the **Auto Scaling Groups** page, on the banner at the top of the page, choose **Go to the new console**\. 

**To create an Auto Scaling group \(new console\)**

1. From the new console, on the **Auto Scaling groups** page, choose **Create an Auto Scaling group**\.

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group\.

1. For **Launch template**, choose one of the following options:

   1. \[Launch template only\] Choose the launch template that you created and then choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

   1. \[Launch configuration only\] Choose **Switch to launch configuration** and choose the launch configuration that you created\.

1. Choose **Next**\. 

   The **Configure settings** page appears, allowing you to configure network settings and giving you options for enabling diversification across On\-Demand Instances and Spot Instances of multiple instance types \(if you chose a launch template\)\. 

1. \[Launch template only\] Keep **Purchase options and instance types** set to **Adhere to the launch template** to use the EC2 instance type and purchase option that are specified in the launch template\. 

1. For **Network**, choose the VPC that you used for your load balancer and security group\.

1. For **Subnet**, choose one or more subnets from each Availability Zone that you want to include\.

1. Choose **Next**\. 

1. On the **Specify load balancing and health checks** page, under **Load balancing**, choose **Enable load balancing**\.

1. Do one of the following:

   1. Choose **Application Load Balancer or Network Load Balancer**, and then choose your load balancer target group\.

   1. Choose **Classic Load Balancers**, and then choose your load balancer\.

1. \(Optional\) To use Elastic Load Balancing health checks, for **Health checks**, choose **ELB** under **Health check type**\.

1. When you have finished configuring the Auto Scaling group, choose **Skip to review**\. 

1. On the **Review** page, review the details of your Auto Scaling group\. You can choose **Edit** to make changes\. When you are finished, choose **Create Auto Scaling group**\.

**To create an Auto Scaling group \(old console\)**

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

1. On the **Configure scaling policies** page, select **Keep this group at its initial size**\.

1. When you have finished configuring the Auto Scaling group, choose **Review**\.

1. Review the details of your Auto Scaling group\. You can choose **Edit** to make changes\. When you are finished, choose **Create Auto Scaling group**\.

After you have created the Auto Scaling group with the load balancer attached, the load balancer automatically registers new instances as they come online\. You have only one instance at this point, so there isn't much to register\. However, you can now add additional instances by updating the desired capacity of the group\. For step\-by\-step instructions, see [Manual Scaling](as-manual-scaling.md)\. 

### \(Optional\) Verify That Your Load Balancer Is Attached to Your Auto Scaling Group<a name="as-register-lbs-verify-console"></a>

**To verify that your load balancer is attached to your Auto Scaling group \(new console\)**

1. Select the check box next to your Auto Scaling group\.

   If you're seeing the old user interface, switch to the new user interface by choosing **Go to the new console** from the banner that's displayed at the top of the console page\.

1. On the **Details** tab, **Load balancing** shows any attached load balancer target groups or Classic Load Balancers\.

1. On the **Activity** tab, under **Activity history**, the **Status** column shows you whether your Auto Scaling group has successfully launched instances\.

1. On the **Instance management** tab, under **Instances**, you can view the status of the instances that are currently running\. 

   The **Lifecycle** column shows the state of your instances\. After an instance is ready to receive traffic, its state is `InService`\.

   The **Health status** column shows the result of the health checks on your instances\.

**To verify that your load balancer is attached to your Auto Scaling group \(old console\)**

1. Select your Auto Scaling group\.

1. On the **Details** tab, **Load Balancers** shows any attached load balancers, and **Target Groups** shows any attached target groups\.

1. On the **Activity History** tab, the **Status** column shows you the status of your Auto Scaling instances\. While an instance is launching, its status is `In progress`\. The status changes to `Successful` after the instance is launched\.

1. On the **Instances** tab, the **Lifecycle** column shows the state of your Auto Scaling instances\. After an instance is ready to receive traffic, its state is `InService`\.

   The **Health Status** column shows the result of the health checks on your instances\.

## Deploy Your Application \(AWS CLI\)<a name="as-lbs-app-cli"></a>

The following sections step you through the process of deploying your application\. 

**Topics**
+ [Create a Launch Template](#as-lbs-app-create-lt-cli)
+ [Create a Launch Configuration](#as-lbs-app-create-lc-cli)
+ [Create an Auto Scaling Group with a Load Balancer](#as-lbs-app-create-asg-cli)

### Create a Launch Template<a name="as-lbs-app-create-lt-cli"></a>

If you already have a launch template that you'd like to use, skip this step\.

**To create the launch template**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command and pass a JSON file that contains the information for creating the launch template\. To assign public IP addresses to instances in a nondefault VPC, add the network interface attribute and set it to `"NetworkInterfaces":{"AssociatePublicIpAddress":true}` as shown\. 

```
aws ec2 create-launch-template --launch-template-name my-launch-template --version-description my-version-description \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"AssociatePublicIpAddress":true,"Groups":["sg-903004f8"],"DeleteOnTermination":true}],"ImageId":"ami-01e24be29428c15b2","InstanceType":"t2.micro"}'
```

### Create a Launch Configuration<a name="as-lbs-app-create-lc-cli"></a>

If you already have a launch configuration that you'd like to use, skip this step\.

**To create the launch configuration**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command\. To assign public IP addresses to instances in a nondefault VPC, use the `--associate-public-ip-address` option as shown\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-launch-config \
  --image-id ami-01e24be29428c15b2 --instance-type t2.micro --associate-public-ip-address \
  --security-groups sg-eb2af88e
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

## Next Steps<a name="as-lbs-app-next-steps"></a>

Now that you have completed this tutorial, you can learn more:
+ You can configure your Auto Scaling group to use a scaling policy\. This allows the group to scale your Amazon EC2 capacity automatically to handle changes in the amount of traffic that your application receives\. For more information, see [Dynamic Scaling](as-scale-based-on-demand.md)\. 
+ If you would like to learn how to create automated schedules for scaling, [Scheduled Scaling](schedule_time.md) provides an introduction to scheduling an Auto Scaling group to scale using one\-time or recurring scaling actions\.
+ You can configure your Auto Scaling group to use Elastic Load Balancing health checks\. If you enable load balancer health checks and an instance fails the health checks, the Auto Scaling group considers the instance unhealthy and replaces it\. For more information, see [Adding ELB Health Checks](as-add-elb-healthcheck.md)\.
+ You can expand your application to an additional Availability Zone in the same AWS Region to increase fault tolerance in the event of a service disruption\. For more information, see [Adding an Availability Zone](as-add-availability-zone.md)\.

## Clean Up Your AWS Resources<a name="as-lbs-app-clean-up"></a>

You've now successfully completed the tutorial\. To avoid unnecessary charges to your account for resources that you aren't using, you should clean up the resources that you created just for this tutorial\. For step\-by\-step instructions, see [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)\.