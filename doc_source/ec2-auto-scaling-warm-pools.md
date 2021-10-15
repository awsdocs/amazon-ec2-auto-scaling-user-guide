# Warm pools for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-warm-pools"></a>

A warm pool gives you the ability to decrease latency for your applications that have exceptionally long boot times, for example, because instances need to write massive amounts of data to disk\. With warm pools, you no longer have to over\-provision your Auto Scaling groups to manage latency in order to improve application performance\. 

This section shows how to add a warm pool to your Auto Scaling group\. A warm pool is a pool of pre\-initialized EC2 instances that sits alongside the Auto Scaling group\. Whenever your application needs to scale out, the Auto Scaling group can draw on the warm pool to meet its new desired capacity\. The goal of a warm pool is to ensure that instances are ready to quickly start serving application traffic, accelerating the response to a scale\-out event\. This is known as a *warm start*\. 

**Important**  
Creating a warm pool when it's not required can lead to unnecessary costs\. If your first boot time does not cause noticeable latency issues for your application, there probably isn't a need for you to use a warm pool\.

You have the option of keeping instances in the warm pool in one of two states: `Stopped` or `Running`\. Keeping instances in a `Stopped` state is an effective way to minimize costs\. With stopped instances, you pay only for the volumes that you use and the Elastic IP addresses that are not assigned to a running instance\. But you don't pay for the stopped instances themselves\. You pay for the instances only when they are running\. 

The size of the warm pool is calculated as the difference between two numbers: the Auto Scaling group's maximum capacity and its desired capacity\. For example, if the desired capacity of your Auto Scaling group is 6 and the maximum capacity is 10, the size of your warm pool will be 4 when you first set up the warm pool and the pool is initializing\. The exception is if you specify a value for **Max prepared capacity**, in which case the size of the warm pool is calculated as the difference between the **Max prepared capacity** and the desired capacity\. For example, if the desired capacity of your Auto Scaling group is 6, if the maximum capacity is 10, and if the **Max prepared capacity** is 8, the size of your warm pool will be 2 when you first set up the warm pool and the pool is initializing\. 

As instances leave the warm pool, they count toward the desired capacity of the group\. There are two scenarios in which instances in the warm pool are replenished: when the group scales in and its desired capacity decreases, or when the group scales out and the minimum size of the warm pool is reached\.

Lifecycle hooks control when new instances transition to and from the warm pool\. These hooks help you make sure that your instances are fully configured for your application before they are added to the warm pool at the end of the lifecycle hook\. On the next scale\-out event, the instances are then ready to be added to the Auto Scaling group as soon as possible\. Lifecycle hooks can also help you to ensure that instances are ready to serve traffic as they are leaving the warm pool \(for example, by populating their cache\)\. For more information, see [Warm pool instance lifecycle](warm-pool-instance-lifecycle.md)\.

You can use the AWS Management Console, the AWS CLI, or one of the SDKs to add a warm pool to an Auto Scaling group\. 

**Topics**
+ [Before you begin](#warm-pool-prerequisites)
+ [Add a warm pool \(console\)](#add-warm-pool-console)
+ [Add a warm pool \(AWS CLI\)](#add-warm-pool-aws-cli)
+ [Updating instances in a warm pool](#update-warm-pool)
+ [Delete a warm pool](#delete-warm-pool)
+ [Limitations](#warm-pools-limitations)
+ [Warm pool instance lifecycle](warm-pool-instance-lifecycle.md)
+ [Event types and event patterns that you use when you add or update lifecycle hooks](warm-pools-eventbridge-events.md)
+ [Creating EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md)
+ [Viewing health check status and the reason for health check failures](warm-pools-health-checks-monitor-view-status.md)

## Before you begin<a name="warm-pool-prerequisites"></a>

Decide how you will use lifecycle hooks to prepare the instances for use\. For example, you can configure lifecycle hooks to delay instances from being added to the warm pool before they complete their user data script\. Another way to use lifecycle hooks is to run code in a Lambda function triggered by lifecycle events\. For more information, see the following blog post [Scaling your applications faster with EC2 Auto Scaling Warm Pools](http://aws.amazon.com/blogs/compute/scaling-your-applications-faster-with-ec2-auto-scaling-warm-pools/)\.

Consider adjusting the warm\-up time in your target tracking or step scaling policy to the minimum amount of time necessary\. As instances leave the warm pool, Amazon EC2 Auto Scaling doesn't count them towards the aggregated CloudWatch metrics of the Auto Scaling group until after the lifecycle hook finishes and the warm\-up time completes\. If the warm\-up time delays an `InService` instance from being included in the aggregated group metrics, you might find that your scaling policy scales out more aggressively than necessary\. For more information, see [Dynamic scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)\.

## Add a warm pool \(console\)<a name="add-warm-pool-console"></a>

The following procedure demonstrates the steps for adding a warm pool to an Auto Scaling group\.

**To add a warm pool**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page\. 

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

## Add a warm pool \(AWS CLI\)<a name="add-warm-pool-aws-cli"></a>

The following examples show how to add a warm pool with the AWS CLI [put\-warm\-pool](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-warm-pool.html) command\. Each example shows how you can specify the command in each of the example scenarios\.

**Topics**
+ [Example 1: Keeping instances in the `Stopped` state](#warm-pool-configuration-ex1)
+ [Example 2: Keeping instances in the `Running` state](#warm-pool-configuration-ex2)
+ [Example 3: Specifying the minimum number of instances in the warm pool](#warm-pool-configuration-ex3)
+ [Example 4: Defining a warm pool size separate from the maximum group size](#warm-pool-configuration-ex4)
+ [Example 5: Defining an absolute warm pool size](#warm-pool-configuration-ex5)

### Example 1: Keeping instances in the `Stopped` state<a name="warm-pool-configuration-ex1"></a>

The following warm pool configuration is for a warm pool that keeps instances in a `Stopped` state\. While instances are stopped, you don't have to pay any EC2 instance usage fees, though you are charged for any other resources that are attached to your stopped instances such as EBS volumes and Elastic IP addresses\.

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped
```

### Example 2: Keeping instances in the `Running` state<a name="warm-pool-configuration-ex2"></a>

The following example configuration is for a warm pool that keeps instances in a `Running` state instead of a `Stopped` state\. You pay for instances while they are running\. The costs incurred are based on the instance type\.

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Running
```

### Example 3: Specifying the minimum number of instances in the warm pool<a name="warm-pool-configuration-ex3"></a>

The following warm pool configuration specifies a minimum of 4 instances in the warm pool, so that this number of instances is always ready to handle traffic spikes\. 

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped --min-size 4
```

### Example 4: Defining a warm pool size separate from the maximum group size<a name="warm-pool-configuration-ex4"></a>

Generally, because you have a good understanding of how much above your desired capacity to set your maximum capacity, there is no need to define an additional maximum size\. Amazon EC2 Auto Scaling simply creates a warm pool that dynamically resizes based on where your maximum capacity is set\. 

However, in cases where you want to have control over the group's maximum capacity and not have it impact the size of the warm pool, you can use the `--max-group-prepared-capacity` option\. You might need to use this option when working with very large Auto Scaling groups to manage the cost benefits of having a warm pool\. For example, an Auto Scaling group with 1000 instances, a maximum capacity of 1500 \(to provide extra capacity for emergency traffic spikes\), and a warm pool of 100 instances might achieve your objectives better than a warm pool of 500 instances\.

The following warm pool configuration is for a warm pool that defines its size separately from the maximum group size\. Suppose the Auto Scaling group has a desired capacity of 800\. The size of the warm pool will be 100 when you run this command and the pool is initializing\.

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped --max-group-prepared-capacity 900
```

To maintain a minimum number of instances in the warm pool, include the `--min-size` option with the command, as follows\. 

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped --max-group-prepared-capacity 900 --min-size 25
```

### Example 5: Defining an absolute warm pool size<a name="warm-pool-configuration-ex5"></a>

If you set the values for the `--max-group-prepared-capacity` and `--min-size` options to the same value, the warm pool will have an absolute size\. The following warm pool configuration is for a warm pool that maintains a constant warm pool size of 10 instances\.

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped --min-size 10 --max-group-prepared-capacity 10
```

## Updating instances in a warm pool<a name="update-warm-pool"></a>

To change the launch template or launch configuration for a warm pool, update the Auto Scaling group to use the new launch template or launch configuration\. After you change the launch template or launch configuration for an Auto Scaling group, any new instances are launched using the new configuration options, but existing instances are not affected\.

To force replacement warm pool instances to launch that use the new configuration options, you can terminate existing instances in the warm pool\. Amazon EC2 Auto Scaling immediately starts launching new instances to replace the instances that you terminated\. Alternatively, you can start an instance refresh to do a rolling update of your group\. An instance refresh first replaces `InService` instances\. Then it replaces instances in the warm pool\. For more information, see [Replacing Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

## Delete a warm pool<a name="delete-warm-pool"></a>

When you no longer need the warm pool, use the following procedure to delete it\.

**To delete your warm pool \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Delete**\. 

**To delete your warm pool \(AWS CLI\)**  
Use the following [delete\-warm\-pool](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-warm-pool.html) command to delete the warm pool\. 

```
aws autoscaling delete-warm-pool --auto-scaling-group-name my-asg
```

If there are instances in the warm pool, or if scaling activities are in progress, use the [delete\-warm\-pool](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-warm-pool.html) command with the `--force-delete` option\. This option will also terminate the Amazon EC2 instances and any outstanding lifecycle actions\.

```
aws autoscaling delete-warm-pool --auto-scaling-group-name my-asg --force-delete
```

## Limitations<a name="warm-pools-limitations"></a>
+ You cannot add a warm pool to Auto Scaling groups that have a mixed instances policy or that launch Spot Instances\.
+ You can put an instance in a `Stopped` state only if it has an Amazon EBS volume as its root device\. Instances that use instance stores for the root device cannot be stopped\.
+ If your warm pool is depleted when there is a scale\-out event, instances will launch directly into the Auto Scaling group \(a *cold start*\)\. You could also experience cold starts if an Availability Zone is out of capacity\.
+ If you try using warm pools with Amazon Elastic Container Service \(Amazon ECS\) or Elastic Kubernetes Service \(Amazon EKS\) managed node groups, there is a chance that these services will schedule jobs on an instance before it reaches the warm pool\.