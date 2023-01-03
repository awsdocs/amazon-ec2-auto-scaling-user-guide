# Set the health check grace period for an Auto Scaling group<a name="health-check-grace-period"></a>

The health check grace period specifies the minimum amount of time \(in seconds\) to keep a new instance in service before terminating if it's unhealthy\. A specific use case might be a requirement to wait to act on Elastic Load Balancing health checks because the instance is still initializing\. The grace period prevents Amazon EC2 Auto Scaling from marking your newly launched instances unhealthy and terminating them unnecessarily if they do not immediately pass these health checks after they enter the `InService` state\.

In the console, by default, the health check grace period is 300 seconds when you create an Auto Scaling group\. Its default value is 0 seconds when you create an Auto Scaling group using the AWS CLI or an SDK\. 

Setting this value too high reduces the effectiveness of the Amazon EC2 Auto Scaling health checks\. If you use lifecycle hooks for instance launch, you can set the health check grace period to 0\. With lifecycle hooks, Amazon EC2 Auto Scaling provides a way to make sure that instances are always initialized before they enter the `InService` state\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

The grace period applies to the following instances:
+ Newly launched instances
+ Instances that are put back into service after being in standby
+ Instances that you manually attach to the group

**Important**  
During the health check grace period, if Amazon EC2 Auto Scaling detects that an instance is no longer in the Amazon EC2 `running` state, it immediately marks the instance as unhealthy and replaces it\. For example, if you stop an instance in an Auto Scaling group, it is marked unhealthy and replaced\.

## Set the health check grace period for a group<a name="set-health-check-grace-period"></a>

You can set the health check grace period for new and existing Auto Scaling groups\.

------
#### [ Console ]

**To modify the health check grace period for a new group \(console\)**  
When you create the Auto Scaling group, on the **Configure advanced options** page, for **Health checks**, **Health check grace period**, enter the amount of time, in seconds\. This is how long Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance after it enters the `InService` state\.

------
#### [ AWS CLI ]

**To modify the health check grace period for a new group \(AWS CLI\)**  
Add the `--health-check-grace-period` option to the [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command\. The following example configures the health check grace period with a value of `60` seconds for a new Auto Scaling group named `my-asg`\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg --health-check-grace-period 60 ...
```

------

------
#### [ Console ]

**To modify the health check grace period for an existing group \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. On the navigation bar at the top of the screen, choose the AWS Region that you created your Auto Scaling group in\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. Under **Health check grace period**, enter the amount of time, in seconds\. This is how long Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance after it enters the `InService` state\.

1. Choose **Update**\.

------
#### [ AWS CLI ]

**To modify the health check grace period for an existing group \(AWS CLI\)**  
Add the `--health-check-grace-period` option to the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\. The following example configures the health check grace period with a value of `120` seconds for an existing Auto Scaling group named `my-asg`\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --health-check-grace-period 120
```

------

**Note**  
We strongly recommend also setting the default instance warm\-up time for your Auto Scaling group\. For a consolidated view of all your instance warm\-up, health check grace period, and cooldown settings in one place, see [Available warm\-up and cooldown settings](consolidated-view-of-warm-up-and-cooldown-settings.md)\.