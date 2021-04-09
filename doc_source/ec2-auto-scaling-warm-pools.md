# Warm pools for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-warm-pools"></a>

A warm pool gives you the ability to decrease latency for your applications that have exceptionally long boot times, for example, because instances need to write massive amounts of data to disk\. With warm pools, you no longer have to over\-provision your Auto Scaling groups to manage latency in order to improve application performance\. 

This section shows how to add a warm pool to your Auto Scaling group\. A warm pool is a pool of pre\-initialized EC2 instances that sits alongside the Auto Scaling group\. Whenever your application needs to scale out, the Auto Scaling group can draw on the warm pool to meet its new desired capacity\. The goal of a warm pool is to ensure that instances are ready to quickly start serving application traffic, accelerating the response to a scale\-out event\. This is known as a *warm start*\. 

**Important**  
Creating a warm pool when it's not required can lead to unnecessary costs\. If your first boot time does not cause noticeable latency issues for your application, there probably isn't a need for you to use a warm pool\.

You have the option of keeping instances in the warm pool in one of two states: `Stopped` or `Running`\. Keeping instances in a `Stopped` state is an effective way to minimize costs\. With stopped instances, you pay only for the volumes that you use and the Elastic IP addresses that are not assigned to a running instance\. But you don't pay for the stopped instances themselves\. You pay for the instances only when they are running\. 

The size of the warm pool is calculated as the difference between two numbers: the Auto Scaling group's maximum capacity and its desired capacity\. For example, if the desired capacity of your Auto Scaling group is 6 and the maximum capacity is 10, the size of your warm pool will be 4 when you first set up the warm pool and the pool is initializing\. The exception is if you specify a value for **Max prepared capacity**, in which case the size of the warm pool is calculated as the difference between the **Max prepared capacity** and the desired capacity\. For example, if the desired capacity of your Auto Scaling group is 6, if the maximum capacity is 10, and if the **Max prepared capacity** is 8, the size of your warm pool will be 2 when you first set up the warm pool and the pool is initializing\. 

As instances leave the warm pool, they count toward the desired capacity of the group\. There are two scenarios in which instances in the warm pool are replenished: when the group scales in and its desired capacity decreases, or when the group scales out and the minimum size of the warm pool is reached\.

Lifecycle hooks control when new instances transition to and from the warm pool\. These hooks help you make sure that your instances are fully configured for your application before they are added to the warm pool at the end of the lifecycle hook\. On the next scale\-out event, the instances are then ready to be added to the Auto Scaling group as soon as possible\. Lifecycle hooks can also help you to ensure that instances are ready to serve traffic as they are leaving the warm pool \(for example, by populating their cache\)\. For more information, see [Warm pool instance lifecycle](warm-pool-instance-lifecycle.md)\.

You can use the AWS Management Console, the AWS CLI, or an AWS SDK to add a warm pool to an Auto Scaling group\. 

**Topics**
+ [Before you begin](#warm-pool-prerequisites)
+ [Add a warm pool \(console\)](#add-warm-pool-console)
+ [Add a warm pool \(AWS CLI\)](#add-warm-pool-aws-cli)
+ [Limitations](#warm-pools-limitations)
+ [Warm pool instance lifecycle](warm-pool-instance-lifecycle.md)
+ [Event types and event patterns that you use when you add or update lifecycle hooks](warm-pools-eventbridge-events.md)
+ [Creating EventBridge rules for warm pool events](warm-pool-events-eventbridge-rules.md)
+ [Viewing health check status and the reason for health check failures](warm-pools-health-checks-monitor-view-status.md)
+ [Deleting a warm pool](delete-warm-pool.md)

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

      If you choose **Define a set number of instances**, you must enter the maximum instance count\. This value represents the maximum number of instances that are allowed to be in the warm pool and the Auto Scaling group at the same time\.
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
aws aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped --min-size 4
```

### Example 4: Defining a warm pool size separate from the maximum group size<a name="warm-pool-configuration-ex4"></a>

Generally, because you have a good understanding of how much above your desired capacity to set your maximum capacity, there is no need to define an additional maximum size\. Amazon EC2 Auto Scaling simply creates a warm pool that dynamically resizes based on where your maximum capacity is set\. 

However, in cases where you want to have control over the group's maximum size and not have it impact the size of the warm pool, you can use the `--max-group-prepared-capacity` option\. You might need to use this option when working with very large Auto Scaling groups to manage the cost benefits of having a warm pool\. For example, an Auto Scaling group with 1000 instances, a maximum capacity of 1500 \(to provide extra capacity for emergency traffic spikes\), and a warm pool of 100 instances might achieve your objectives better than a warm pool of 500 instances\.

The following warm pool configuration is for a warm pool that defines its size separately from the maximum group size\. Suppose the Auto Scaling group has a desired capacity of 800\. The size of the warm pool will be 100 when you run this command and the pool is initializing\. 

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped --max-group-prepared-capacity 900
```

### Example 5: Defining an absolute warm pool size<a name="warm-pool-configuration-ex5"></a>

If you set the values for the `--max-group-prepared-capacity` and `--min-size` options to the same value, the warm pool will have an absolute size\. The following warm pool configuration is for a warm pool that maintains a constant warm pool size of 10 instances\.

```
aws autoscaling put-warm-pool --auto-scaling-group-name my-asg /
  --pool-state Stopped --min-size 10 --max-group-prepared-capacity 10
```

## Limitations<a name="warm-pools-limitations"></a>
+ You cannot add a warm pool to Auto Scaling groups that have a mixed instances policy or that launch Spot Instances\.
+ You can put an instance in a `Stopped` state only if it has an Amazon EBS volume as its root device\. Instances that use instance stores for the root device cannot be stopped\.
+ If your warm pool is depleted when there is a scale out event, instances will launch directly into the Auto Scaling group \(a *cold start*\)\. You could also experience cold starts if an Availability Zone is out of capacity\.
+ If you set **Max prepared capacity**, and the desired capacity of the Auto Scaling group is higher than the **Max prepared capacity**, the capacity of the warm pool is 0\. 
+ Warm pools currently can't be used with ECS, EKS, and self\-managed Kubernetes\.