# Creating an Auto Scaling group using the Amazon EC2 launch wizard<a name="create-asg-ec2-wizard"></a>

The following procedure shows how to create an Auto Scaling group by using the **Launch instance** wizard in the Amazon EC2 console\. This option automatically populates a launch template with certain configuration details from the **Launch instance** wizard\.

**Note**  
The wizard does not populate the Auto Scaling group with the number of instances you specify; it only populates the launch template with the Amazon Machine Image \(AMI\) ID and instance type\. Use the **Create Auto Scaling group** wizard to specify the number of instances to launch\.   
An AMI provides the information required to configure an instance\. You can launch multiple instances from a single AMI when you need multiple instances with the same configuration\. We recommend using a custom AMI that already has your application installed on it to avoid having your instances terminated if you reboot an instance belonging to an Auto Scaling group\. To use a custom AMI with Amazon EC2 Auto Scaling, you must first create your AMI from a customized instance, and then use the AMI to create a launch template for your Auto Scaling group\.

**Prerequisites**
+ You must have created a custom AMI in the same AWS Region where you plan to create the Auto Scaling group\. For more information, see [Create an AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-ami.html) in the *Amazon EC2 User Guide for Linux Instances*\.
+ You must have IAM permissions to create an Auto Scaling group using a launch template and also to create EC2 resources for the instances\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

**To use a custom AMI as a template**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar at the top of the screen, the current AWS Region is displayed\. Select a Region in which to launch your Auto Scaling group\.

1. From the console dashboard, choose **Launch instance**, choose **Launch instance** again, and then do the following:

   1. On the **Choose an Amazon Machine Image \(AMI\)** page, choose **My AMIs**, find the AMI that you created, and then choose **Select**\.

   1. On the **Choose an Instance Type** page, choose an instance type\. 
**Note**  
Select the same instance type that you used when you created the AMI or a more powerful one\.

   1. At the bottom of the screen, choose **Next: Configure Instance Details**\.

   1. In **Step 3: Configure Instance Details**, choose **Launch into Auto Scaling Group** next to **Number of instances**\.

   1. On the confirmation dialogue, choose **Continue** to go to the **Create launch template** page with the AMI and instance type you selected in the launch instance wizard already populated\.

## Create a launch template<a name="create-asg-ec2-wizard-launch-template"></a>

After you choose **Continue**, the **Create launch template** page opens\. Follow this procedure to create a launch template\. For more information, see [Creating your launch template \(console\)](create-launch-template.md#create-launch-template-for-auto-scaling)\.

**To create a launch template**

1. Enter a name and description for the new launch template\.

1. \(Optional\) For **Key pair \(login\)**, **Key pair name**, choose the name of the previously created key pair to use when connecting to instances, for example, using SSH\.

1. \(Optional\) For **Network settings**, **Security groups**, choose one or more previously created [security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)\.

1. \(Optional\) For **Storage \(Volumes\)**, update the storage configuration\. The default storage configuration is determined by the AMI and the instance type\. 

1. When you are done configuring the launch template, choose **Create launch template**\.

1. On the confirmation page, choose **Create Auto Scaling group**\.

## Create an Auto Scaling group<a name="create-asg-ec2-wizard-auto-scaling-group"></a>

**Note**  
The rest of this topic describes the basic procedure for creating an Auto Scaling group\. For more description of the parameters you can configure for your Auto Scaling group, see [Creating an Auto Scaling group using a launch template](create-asg-launch-template.md)\.

After you choose **Create Auto Scaling group**, the **Create Auto Scaling group** wizard opens\. Follow this procedure to create an Auto Scaling group\.

**To create an Auto Scaling group**

1. On the **Choose launch template or configuration** page, enter a name for the Auto Scaling group\.

1. The launch template that you created is already selected for you\. 

   For **Launch template version**, choose whether the Auto Scaling group uses the default, the latest, or a specific version of the launch template when scaling out\.

1. Choose **Next** to continue to the next step\.

1. On the **Choose instance launch options** page, under **Network**, for **VPC**, choose a VPC\. The Auto Scaling group must be created in the same VPC as the security group you specified in your launch template\.
**Tip**  
If you didn't specify a security group in your launch template, your instances are launched with a default security group from the VPC that you specify\. By default, this security group doesn't allow inbound traffic from external networks\.

1. For **Availability Zones and subnets**, choose one or more subnets in the specified VPC\.

1. Choose **Next** twice to go to the **Configure group size and scaling policies** page\.

1. Under **Group size**, define the **Desired capacity** \(initial number of instances to launch immediately after the Auto Scaling group is created\), **Minimum capacity** \(minimum number of instances\), and **Maximum capacity** \(maximum number of instances\)\.

1. Choose **Skip to review**\. 

1. On the **Review** page, choose **Create Auto Scaling group**\.

## Next steps<a name="create-asg-ec2-wizard-next-steps"></a>

You can check that the Auto Scaling Group has been created correctly by viewing the activity history\. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched instances\. If the instances fail to launch or they launch but then immediately terminate, see the following topics for possible causes and resolutions:
+ [Troubleshooting Amazon EC2 Auto Scaling: EC2 instance launch failures](ts-as-instancelaunchfailure.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: AMI issues](ts-as-ami.md)
+ [Troubleshooting Amazon EC2 Auto Scaling: Health checks](ts-as-healthchecks.md)

You can now attach a load balancer in the same Region as your Auto Scaling group, if desired\. For more information, see [Elastic Load Balancing and Amazon EC2 Auto Scaling](autoscaling-load-balancer.md)\.