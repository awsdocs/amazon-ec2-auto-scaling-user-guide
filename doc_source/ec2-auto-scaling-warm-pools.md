# Warm pools for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-warm-pools"></a>

A warm pool gives you the ability to decrease latency for your applications that have exceptionally long boot times, for example, because instances need to write massive amounts of data to disk\. With warm pools, you no longer have to over\-provision your Auto Scaling groups to manage latency in order to improve application performance\. For more information, see the following blog post [Scaling your applications faster with EC2 Auto Scaling Warm Pools](http://aws.amazon.com/blogs/compute/scaling-your-applications-faster-with-ec2-auto-scaling-warm-pools/)\.

**Important**  
Creating a warm pool when it's not required can lead to unnecessary costs\. If your first boot time does not cause noticeable latency issues for your application, there probably isn't a need for you to use a warm pool\.

**Topics**
+ [Core concepts](#warm-pool-core-concepts)
+ [Prerequisites](#warm-pool-prerequisites)
+ [Create a warm pool](#create-a-warm-pool-console)
+ [Updating instances in a warm pool](#update-warm-pool)
+ [Delete a warm pool](#delete-warm-pool)
+ [Limitations](#warm-pools-limitations)
+ [Using lifecycle hooks](warm-pool-instance-lifecycle.md)
+ [Event types and event patterns](warm-pools-eventbridge-events.md)
+ [Creating EventBridge rules](warm-pool-events-eventbridge-rules.md)
+ [Viewing health check status](warm-pools-health-checks-monitor-view-status.md)
+ [AWS CLI examples for working with warm pools](examples-warm-pools-aws-cli.md)

## Core concepts<a name="warm-pool-core-concepts"></a>

Before you get started, familiarize yourself with the following core concepts:

**Warm pool**  
A warm pool is a pool of pre\-initialized EC2 instances that sits alongside an Auto Scaling group\. Whenever your application needs to scale out, the Auto Scaling group can draw on the warm pool to meet its new desired capacity\. This helps you to ensure that instances are ready to quickly start serving application traffic, accelerating the response to a scale\-out event\. As instances leave the warm pool, they count toward the desired capacity of the group\. This is known as a *warm start*\. 

**Warm pool size**  
By default, the size of the warm pool is calculated as the difference between the Auto Scaling group's maximum capacity and its desired capacity\. For example, if the desired capacity of your Auto Scaling group is 6 and the maximum capacity is 10, the size of your warm pool will be 4 when you first set up the warm pool and the pool is initializing\.   
If you set a value for maximum prepared capacity, the size of the warm pool is calculated as the difference between the maximum prepared capacity and the desired capacity\. For example, if the desired capacity of your Auto Scaling group is 6, if the maximum capacity is 10, and if the maximum prepared capacity is 8, the size of your warm pool will be 2 when you first set up the warm pool and the pool is initializing\. 

**Warm pool instance state**  
You can keep instances in the warm pool in one of two states: `Stopped` or `Running`\. Keeping instances in a `Stopped` state is an effective way to minimize costs\. With stopped instances, you pay only for the volumes that you use and the Elastic IP addresses attached to the instances\. But you don't pay for the stopped instances themselves\. You pay for the instances only when they are running\. 

**Lifecycle hooks**  
[Lifecycle hooks](lifecycle-hooks.md) let you put instances into a wait state so that you can perform custom actions on the instances\. Custom actions are performed as the instances launch or before they terminate\.  
In a warm pool configuration, lifecycle hooks can also delay instances from being stopped and from being put in service during a scale\-out event until they have finished initializing\. If you add a warm pool to your Auto Scaling group without a lifecycle hook, instances that take a long time to finish initializing could be stopped and then put in service during a scale\-out event before they are ready\.

## Prerequisites<a name="warm-pool-prerequisites"></a>

Decide how you will use lifecycle hooks to prepare the instances for use\. There are two ways to perform custom actions on your instances\.
+ For simple scenarios where you want to run commands on your instances at launch, you can include a user data script when you create a launch template or launch configuration for your Auto Scaling group\. User data scripts are just normal shell scripts or cloud\-init directives that are run by [cloud\-init](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#amazon-linux-cloud-init) when your instances start\. The script can also control when your instances transition to the next state by using the ID of the instance on which it runs\. If you are not doing so already, update your script to retrieve the instance ID of the instance from the instance metadata\. For more information, see [Retrieving instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) in the *Amazon EC2 User Guide for Linux Instances*\.
**Tip**  
To run user data scripts when an instance restarts, the user data must be in the MIME multi\-part format and specify the following in the `#cloud-config` section of the user data:  

  ```
  #cloud-config
  cloud_final_modules:
   - [scripts-user, always]
  ```
+ For advanced scenarios where you need a service such as AWS Lambda to do something as instances are entering or leaving the warm pool, you can create a lifecycle hook for your Auto Scaling group and configure the target service to perform custom actions based on lifecycle notifications\. For more information, see [Supported notification targets](warm-pool-instance-lifecycle.md#warm-pools-supported-notification-targets)\.

For more information, see the lifecycle hook examples in our [GitHub repository](https://github.com/aws-samples/amazon-ec2-auto-scaling-group-examples)\. 

**Additional considerations**  
Consider reducing the instance warm\-up time in your scaling policies\. As instances leave the warm pool, they don't count towards the aggregated CloudWatch instance metrics of the Auto Scaling group until after the lifecycle hook finishes and the warm\-up time completes\. Therefore, warm\-up times that are set too high might cause your scaling policies to scale out more aggressively than desired\. For more information, see [Target tracking scaling policies](as-scaling-target-tracking.md) and [Step and simple scaling policies](as-scaling-simple-step.md)\.

## Create a warm pool<a name="create-a-warm-pool-console"></a>

Create a warm pool using the console according to the following instructions\.

Before you begin, confirm that you have created a lifecycle hook for your Auto Scaling group\.

**To create a warm pool \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. Choose the **Instance management** tab\. 

1. Under **Warm pool**, choose **Create warm pool**\. 

1. To configure a warm pool, do the following:

   1. For **Warm pool instance state**, choose which state you want to transition your instances to when they enter the warm pool\. The default is `Stopped`\. 

   1. For **Minimum warm pool size**, enter the minimum number of instances to maintain in the warm pool\.

   1. \(Optional\) For **Max prepared capacity**, choose **Define a set number of instances** if you want to control how much capacity is available in the warm pool\.

      If you choose **Define a set number of instances**, you must enter the maximum instance count\. This value represents the maximum number of instances that are allowed to be in the warm pool and the Auto Scaling group at the same time\. If you enter a value that is less than the group's desired capacity, and you chose not to specify a value for **Minimum warm pool size**, the capacity of the warm pool will be 0\.
**Note**  
Keeping the default **Equal to the Auto Scaling group's maximum capacity** option helps you maintain a warm pool that is sized to match the difference between the Auto Scaling group's maximum capacity and its desired capacity\. To make it easier to manage your warm pool, we recommend that you use the default so that you do not need to remember to adjust your warm pool settings in order to control the size of the warm pool\.

1. Choose **Create**\. 

## Updating instances in a warm pool<a name="update-warm-pool"></a>

To change the launch template or launch configuration for a warm pool, associate a new launch template or launch configuration with the Auto Scaling group\. Any new instances are launched using the new AMI and other updates that are specified in the launch template or launch configuration, but existing instances are not affected\.

To force replacement warm pool instances to launch that use the new launch template or launch configuration, you can terminate existing instances in the warm pool\. Amazon EC2 Auto Scaling immediately starts launching new instances to replace the instances that you terminated\. Alternatively, you can start an instance refresh to do a rolling update of your group\. An instance refresh first replaces `InService` instances\. Then it replaces instances in the warm pool\. For more information, see [Replacing Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

## Delete a warm pool<a name="delete-warm-pool"></a>

When you no longer need the warm pool, use the following procedure to delete it\.

**To delete your warm pool \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Delete**\. 

## Limitations<a name="warm-pools-limitations"></a>
+ You cannot add a warm pool to Auto Scaling groups that have a mixed instances policy or that launch Spot Instances\.
+ You can put an instance in a `Stopped` state only if it has an Amazon EBS volume as its root device\. Instances that use instance stores for the root device cannot be stopped\.
+ If your warm pool is depleted when there is a scale\-out event, instances will launch directly into the Auto Scaling group \(a *cold start*\)\. You could also experience cold starts if an Availability Zone is out of capacity\.
+ If you try using warm pools with Amazon Elastic Container Service \(Amazon ECS\) or Elastic Kubernetes Service \(Amazon EKS\) managed node groups, there is a chance that these services will schedule jobs on an instance before it reaches the warm pool\.