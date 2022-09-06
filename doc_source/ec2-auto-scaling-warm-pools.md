# Warm pools for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-warm-pools"></a>

A warm pool gives you the ability to decrease latency for your applications that have exceptionally long boot times, for example, because instances need to write massive amounts of data to disk\. With warm pools, you no longer have to over\-provision your Auto Scaling groups to manage latency in order to improve application performance\. For more information, see the following blog post [Scaling your applications faster with EC2 Auto Scaling Warm Pools](http://aws.amazon.com/blogs/compute/scaling-your-applications-faster-with-ec2-auto-scaling-warm-pools/)\.

**Important**  
Creating a warm pool when it's not required can lead to unnecessary costs\. If your first boot time does not cause noticeable latency issues for your application, there probably isn't a need for you to use a warm pool\.

**Topics**
+ [Core concepts](#warm-pool-core-concepts)
+ [Prerequisites](#warm-pool-prerequisites)
+ [Create a warm pool](#create-a-warm-pool-console)
+ [Update a warm pool](#update-warm-pool)
+ [Delete a warm pool](#delete-warm-pool)
+ [Limitations](#warm-pools-limitations)
+ [Use lifecycle hooks](warm-pool-instance-lifecycle.md)
+ [View health check status](warm-pools-health-checks-monitor-view-status.md)
+ [AWS CLI examples for working with warm pools](examples-warm-pools-aws-cli.md)

## Core concepts<a name="warm-pool-core-concepts"></a>

Before you get started, familiarize yourself with the following core concepts:

**Warm pool**  
A warm pool is a pool of pre\-initialized EC2 instances that sits alongside an Auto Scaling group\. Whenever your application needs to scale out, the Auto Scaling group can draw on the warm pool to meet its new desired capacity\. This helps you to ensure that instances are ready to quickly start serving application traffic, accelerating the response to a scale\-out event\. As instances leave the warm pool, they count toward the desired capacity of the group\. This is known as a *warm start*\.   
While instances are in the warm pool, your scaling policies only scale out if the metric value from instances that are in the `InService` state is greater than the scaling policy's alarm high threshold \(which is the same as the target utilization of a target tracking scaling policy\)\.

**Warm pool size**  
By default, the size of the warm pool is calculated as the difference between the Auto Scaling group's maximum capacity and its desired capacity\. For example, if the desired capacity of your Auto Scaling group is 6 and the maximum capacity is 10, the size of your warm pool will be 4 when you first set up the warm pool and the pool is initializing\.   
To specify the warm pool's maximum capacity separately, set a value for maximum prepared capacity that is greater than the current capacity of the group\. When you set a value for maximum prepared capacity, the size of the warm pool is calculated as the difference between the maximum prepared capacity and the current desired capacity of the group\. For example, if the desired capacity of your Auto Scaling group is 6, if the maximum capacity is 10, and if the maximum prepared capacity is 8, the size of your warm pool will be 2 when you first set up the warm pool and the pool is initializing\.   
You might only need to use the maximum prepared capacity option when working with large Auto Scaling groups to manage the cost benefits of having a warm pool\. For example, an Auto Scaling group with 1,000 instances, a maximum capacity of 1,500 \(to provide extra capacity for emergency traffic spikes\), and a warm pool of 100 instances might help you achieve your goals better than keeping 500 instances reserved for future use inside the warm pool\.

**Minimum warm pool size**  
Consider using the minimum size setting to statically set the minimum number of instances to maintain in the warm pool\. There is no minimum size set by default\.

**Warm pool instance state**  
You can keep instances in the warm pool in one of three states: `Stopped`, `Running`, or `Hibernated`\. Keeping instances in a `Stopped` state is an effective way to minimize costs\. With stopped instances, you pay only for the volumes that you use and the Elastic IP addresses attached to the instances\.  
Alternatively, you can keep instances in a `Hibernated` state to stop instances without deleting their memory contents \(RAM\)\. When an instance is hibernated, this signals the operating system to save the contents of your RAM to your Amazon EBS root volume\. When the instance is started again, the root volume is restored to its previous state and the RAM contents are reloaded\. While the instances are in hibernation, you pay only for the EBS volumes, including storage for the RAM contents, and the Elastic IP addresses attached to the instances\.  
Keeping instances in a `Running` state inside the warm pool is also possible, but is highly discouraged to avoid incurring unnecessary charges\. When instances are stopped or hibernated, you are saving the cost of the instances themselves\. You pay for the instances only when they are running\.

**Lifecycle hooks**  
[Lifecycle hooks](lifecycle-hooks.md) let you put instances into a wait state so that you can perform custom actions on the instances\. Custom actions are performed as the instances launch or before they terminate\.  
In a warm pool configuration, lifecycle hooks can also delay instances from being stopped or hibernated and from being put in service during a scale\-out event until they have finished initializing\. If you add a warm pool to your Auto Scaling group without a lifecycle hook, instances that take a long time to finish initializing could be stopped or hibernated and then put in service during a scale\-out event before they are ready\.

**Instance reuse policy**  
By default, Amazon EC2 Auto Scaling terminates your instances when your Auto Scaling group scales in\. Then, it launches new instances into the warm pool to replace the instances that were terminated\.   
If you want to return instances to the warm pool instead, you can specify an instance reuse policy\. This lets you reuse instances that are already configured to serve application traffic\. To make sure that your warm pool is not over\-provisioned, Amazon EC2 Auto Scaling can terminate instances in the warm pool to reduce its size when it is larger than necessary based on its settings\. When terminating instances in the warm pool, it uses the [default termination policy](ec2-auto-scaling-termination-policies.md#default-termination-policy) to choose which instances to terminate first\.   
If you want to hibernate instances on scale in and there are existing instances in the Auto Scaling group, they must meet the requirements for instance hibernation\. If they don't, when instances return to the warm pool, they will fallback to being stopped instead of being hibernated\.
Currently, you can only specify an instance reuse policy by using the AWS CLI or an SDK\. This feature is not available from the console\.

## Prerequisites<a name="warm-pool-prerequisites"></a>

Decide how you will use lifecycle hooks to prepare the instances for use\. There are two ways to perform custom actions on your instances\.
+ For simple scenarios where you want to run commands on your instances at launch, you can include a user data script when you create a launch template or launch configuration for your Auto Scaling group\. User data scripts are just normal shell scripts or cloud\-init directives that are run by [cloud\-init](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#amazon-linux-cloud-init) when your instances start\. The script can also control when your instances transition to the next state by using the ID of the instance on which it runs\. If you are not doing so already, update your script to retrieve the instance ID of the instance from the instance metadata\. For more information, see [Retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) in the *Amazon EC2 User Guide for Linux Instances*\.
**Tip**  
To run user data scripts when an instance restarts, the user data must be in the MIME multi\-part format and specify the following in the `#cloud-config` section of the user data:  

  ```
  #cloud-config
  cloud_final_modules:
   - [scripts-user, always]
  ```
+ For advanced scenarios where you need a service such as AWS Lambda to do something as instances are entering or leaving the warm pool, you can create a lifecycle hook for your Auto Scaling group and configure the target service to perform custom actions based on lifecycle notifications\. For more information, see [Supported notification targets](warm-pool-instance-lifecycle.md#warm-pools-supported-notification-targets)\.

For more information, see the lifecycle hook examples in our [GitHub repository](https://github.com/aws-samples/amazon-ec2-auto-scaling-group-examples)\. 

**Prepare instances for hibernation**  
To prepare Auto Scaling instances to use the `Hibernated` pool state, create a new launch template or launch configuration that is set up correctly to support instance hibernation, as described in the [Hibernation prerequisites](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hibernating-prerequisites.html) topic in the *Amazon EC2 User Guide for Linux Instances*\. Then, associate the new launch template or launch configuration with the Auto Scaling group and start an instance refresh to replace the instances associated with a previous launch template or launch configuration\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

## Create a warm pool<a name="create-a-warm-pool-console"></a>

Create a warm pool using the console according to the following instructions\.

Before you begin, confirm that you have created a lifecycle hook for your Auto Scaling group\.

**To create a warm pool \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to an existing group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. Choose the **Instance management** tab\. 

1. Under **Warm pool**, choose **Create warm pool**\. 

1. To configure a warm pool, do the following:

   1. For **Warm pool instance state**, choose which state you want to transition your instances to when they enter the warm pool\. The default is `Stopped`\. 

   1. For **Minimum warm pool size**, enter the minimum number of instances to maintain in the warm pool\.

   1. For **Max prepared capacity**, you can specify the maximum prepared capacity by defining a set number of instances, or keep the default option to keep the maximum prepared capacity undefined\. 

      If you keep the default, **Equal to the Auto Scaling group's maximum capacity**, the warm pool size is sized to match the difference between the Auto Scaling group's maximum capacity and its desired capacity\. To make it easier to manage the warm pool's size by adjusting the group's maximum capacity, we recommend that you use the default option\. 

      If you choose the **Define a set number of instances** option, enter a value that represents the maximum number of instances that are allowed to be in the warm pool and the Auto Scaling group at the same time\.

1. Choose **Create**\. 

## Update a warm pool<a name="update-warm-pool"></a>

To change the launch template or launch configuration for a warm pool, associate a new launch template or launch configuration with the Auto Scaling group\. Any new instances are launched using the new AMI and other updates that are specified in the launch template or launch configuration, but existing instances are not affected\.

To force replacement warm pool instances to launch that use the new launch template or launch configuration, you can terminate existing instances in the warm pool\. Amazon EC2 Auto Scaling immediately starts launching new instances to replace the instances that you terminated\. Alternatively, you can start an instance refresh to do a rolling update of your group\. An instance refresh first replaces `InService` instances\. Then it replaces instances in the warm pool\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

## Delete a warm pool<a name="delete-warm-pool"></a>

When you no longer need the warm pool, use the following procedure to delete it\.

**To delete your warm pool \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to an existing group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. Choose the **Instance management** tab\. 

1. For **Warm pool**, choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Delete**\. 

## Limitations<a name="warm-pools-limitations"></a>
+ You cannot add a warm pool to Auto Scaling groups that have a mixed instances policy or that launch Spot Instances\.
+ Amazon EC2 Auto Scaling can put an instance in a `Stopped` or `Hibernated` state only if it has an Amazon EBS volume as its root device\. Instances that use instance stores for the root device cannot be stopped or hibernated\.
+ Amazon EC2 Auto Scaling can put an instance in a `Hibernated` state only if meets all of the requirements listed in the [Hibernation prerequisites](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hibernating-prerequisites.html) topic in the *Amazon EC2 User Guide for Linux Instances*\. 
+ If your warm pool is depleted when there is a scale\-out event, instances will launch directly into the Auto Scaling group \(a *cold start*\)\. You could also experience cold starts if an Availability Zone is out of capacity\.
+ If you try using a warm pool with an Amazon Elastic Kubernetes Service \(Amazon EKS\) managed node group, instances that are still initializing might register with your Amazon EKS cluster\. As a result, the cluster might schedule jobs on an instance as it is preparing to be stopped or hibernated\.
+ Likewise, if you try using a warm pool with an Amazon ECS cluster, instances might register with the cluster before they finish initializing\. To solve this problem, you must configure a launch template or launch configuration that includes a special agent configuration variable in the user data\. For more information, see [Using a warm pool for your Auto Scaling group](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/asg-capacity-providers-create-auto-scaling-group.html#using-warm-pool) in the *Amazon Elastic Container Service Developer Guide*\.
+ Hibernation support for warm pools is available in all commercial AWS Regions where Amazon EC2 Auto Scaling is available, excluding the Middle East \(UAE\), China \(Beijing\), China \(Ningxia\), and AWS GovCloud \(US\-East and US\-West\) Regions\.