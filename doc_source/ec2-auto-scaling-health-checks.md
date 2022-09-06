# Health checks for Auto Scaling instances<a name="ec2-auto-scaling-health-checks"></a>

The health status of an Auto Scaling instance indicates whether it is healthy or unhealthy\. All instances in your Auto Scaling group start in the healthy state\. Instances are assumed to be healthy unless Amazon EC2 Auto Scaling receives notification that they are unhealthy\. This notification can come from sources such as Amazon EC2, Elastic Load Balancing, or custom health checks\. When Amazon EC2 Auto Scaling detects an unhealthy instance, it terminates it and launches a new one\. 

**Topics**
+ [Health check types](#available-health-checks)
+ [Amazon EC2 health checks](#instance-health-detection)
+ [Elastic Load Balancing health checks](#elastic-load-balancing-health-checks)
+ [Custom health detection tasks](#as-configure-healthcheck)
+ [Unhealthy instance replacement](#replace-unhealthy-instance)
+ [How Amazon EC2 Auto Scaling minimizes downtime](#minimize-downtime)
+ [Health check considerations](#health-check-considerations)
+ [Health check grace period](#health-check-grace-period)
+ [Additional information](#health-checks-additional-information)

## Health check types<a name="available-health-checks"></a>

Amazon EC2 Auto Scaling can determine the health status of an instance by using one or more of the following health checks:


****  

| Health check type | What it checks | 
| --- | --- | 
| Amazon EC2 status checks and scheduled events |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-health-checks.html) This is the default health check type for an Auto Scaling group\.   | 
| Elastic Load Balancing health checks |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-health-checks.html) To run this health check type, you must enable it for your Auto Scaling group\. | 
| Custom health checks |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-health-checks.html)  | 

## Amazon EC2 health checks<a name="instance-health-detection"></a>

After an instance launches, it is attached to the Auto Scaling group and enters the `InService` state\. For more information about the different lifecycle states for instances in an Auto Scaling group, see [Amazon EC2 Auto Scaling instance lifecycle](ec2-auto-scaling-lifecycle.md)\.

Amazon EC2 Auto Scaling periodically checks the health status of all instances within the Auto Scaling group to make sure that they're running and in good condition\. 

**Status checks**  
Amazon EC2 Auto Scaling uses the results of the Amazon EC2 instance status checks and system status checks to determine the health status of an instance\. If the instance is in any Amazon EC2 state other than `running`, or if its status for the status checks becomes `impaired`, Amazon EC2 Auto Scaling considers the instance to be unhealthy and replaces it\. This includes when the instance has any of the following states:
+ `stopping`
+ `stopped`
+ `shutting-down`
+ `terminated`

The Amazon EC2 status checks do not require any special configuration and are always enabled\. For more information, see [Types of status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html#types-of-instance-status-checks) in the *Amazon EC2 User Guide for Linux Instances*\. 

**Important**  
Amazon EC2 Auto Scaling lets the status checks fail occasionally, without taking any action\. When a status check fails, Amazon EC2 Auto Scaling waits a few minutes for AWS to fix the issue\. It does not immediately mark an instance as unhealthy when its status for the status checks becomes `impaired`\.  
However, if Amazon EC2 Auto Scaling detects that an instance is no longer in the `running` state, this situation is treated as an immediate failure\. In this case, it immediately marks the instance as unhealthy and replaces it\. 

**Scheduled events**  
Amazon EC2 can occasionally schedule events on your instances to be run after a particular timestamp\. For more information, see [Scheduled events for your instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-instances-status-check_sched.html) in the *Amazon EC2 User Guide for Linux Instances*\.

If one of your instances is affected by a scheduled event, Amazon EC2 Auto Scaling considers the instance to be unhealthy and replaces it\. The instance doesn't start terminating until the date and time specified in the timestamp is reached\.

## Elastic Load Balancing health checks<a name="elastic-load-balancing-health-checks"></a>

When you enable Elastic Load Balancing health checks for your Auto Scaling group, Amazon EC2 Auto Scaling can use the results of those health checks to determine the health status of an instance\.

Before you can enable Elastic Load Balancing health checks for your Auto Scaling group, you must do the following:
+ Set up an Elastic Load Balancing load balancer and configure a health check for it to use to determine if your instances are healthy\.
+ Attach the load balancer to your Auto Scaling group\.

After you complete the preceding actions, the following occurs: 
+ Amazon EC2 Auto Scaling registers the instances in the Auto Scaling group with the load balancer\.
+ After an instance finishes registering, it enters the `InService` state and becomes available for use with the load balancer\.

By default, Amazon EC2 Auto Scaling ignores the results of the Elastic Load Balancing health checks\. However, you can enable these health checks for your Auto Scaling group\. After you do this, when Elastic Load Balancing reports a registered instance as `unhealthy`, Amazon EC2 Auto Scaling marks the instance as `unhealthy` on its next periodic health check and replaces it\.

If connection draining \(deregistration delay\) is enabled for your load balancer, Amazon EC2 Auto Scaling waits for either in\-flight requests to complete or the maximum timeout to expire before it terminates unhealthy instances\. 

For more information about how to enable Elastic Load Balancing health checks for your Auto Scaling group, see [Add Elastic Load Balancing health checks to an Auto Scaling group](as-add-elb-healthcheck.md)\.

**Note**  
When you enable Elastic Load Balancing health checks for a group, Amazon EC2 Auto Scaling can terminate and replace instances that are reported as unhealthy, but only after the load balancer is in the `InService` state\. For more information, see [Understand the attachment status of your load balancer](load-balancer-status.md)\.

## Custom health detection tasks<a name="as-configure-healthcheck"></a>

You might also want to run custom health detection tasks on the instances in your Auto Scaling group and set the health status of an instance as unhealthy if the task fails\. This extends your health checks by using a combination of custom health checks, Amazon EC2 status checks, and Elastic Load Balancing health checks, if enabled\.

You can send the instance health information directly to Amazon EC2 Auto Scaling using the AWS CLI or an SDK\. The following examples show how to use the AWS CLI to configure the health state of an instance and then verify the instance's health state\.

Use the following [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command to set the health state of the specified instance to `Unhealthy`\.

```
aws autoscaling set-instance-health --instance-id i-1234567890abcdef0 --health-status Unhealthy
```

Use the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the instance state is `Unhealthy`\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names my-asg
```

The following is an example response that shows that the health status of the instance is `Unhealthy` and that the instance is terminating\.

```
{
    "AutoScalingGroups": [
        {
            ....
            "Instances": [
                {
                    "ProtectedFromScaleIn": false,
                    "AvailabilityZone": "us-west-2a",
                    "LaunchTemplate": {
                        "LaunchTemplateName": "my-launch-template",
                        "Version": "1",
                        "LaunchTemplateId": "lt-1234567890abcdef0"
                    },
                    "InstanceId": "i-1234567890abcdef0",
                    "HealthStatus": "Unhealthy",
                    "LifecycleState": "Terminating"
                },
                ...
            ]
        }
    ]
}
```

## Unhealthy instance replacement<a name="replace-unhealthy-instance"></a>

When Amazon EC2 Auto Scaling determines that an `InService` instance is unhealthy, it terminates the instance while it launches a new replacement instance\. The new instance launches using the current settings of the Auto Scaling group and its associated launch template or launch configuration\.

Amazon EC2 Auto Scaling creates a new scaling activity for terminating the unhealthy instance and then terminates it\. While the instance is terminating, another scaling activity launches a new instance\. 

**To view the reason for health check failures \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to the Auto Scaling group\. 

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched or terminated instances\.

   If it terminated any unhealthy instances, the **Cause** column shows the date and time of the termination and the reason for the health check failure\. For example, `At 2022-05-14T20:11:53Z an instance was taken out of service in response to an ELB system health check failure`\. 

## How Amazon EC2 Auto Scaling minimizes downtime<a name="minimize-downtime"></a>

Health check replacements require instances to terminate first, which might prevent new requests from being accepted until new instances launch\. 

If Amazon EC2 Auto Scaling determines that any instances are no longer running or were marked unhealthy with the [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command, it immediately replaces them\. However, if other instances are found to be unhealthy, Amazon EC2 Auto Scaling uses the following approach to recover from failures\. This approach minimizes any downtime that might occur because of temporary issues or misconfigured health checks\. 
+ If a scaling activity is in progress and your Auto Scaling group is below its desired capacity by 10 percent or more, Amazon EC2 Auto Scaling waits for the in\-progress scaling activity before replacing the unhealthy instances\.
+ When scaling out, Amazon EC2 Auto Scaling waits for the instances to pass an initial health check\. It also waits for the default instance warmup to finish to make sure that the new instances are ready\.
+ After the instances finish warming up and the group has risen to above 90 percent of its desired capacity, Amazon EC2 Auto Scaling replaces the unhealthy instances as follows: 
  + Amazon EC2 Auto Scaling only replaces up to 10 percent of the group's desired capacity at a time\. It does this until all of the unhealthy instances are replaced\. 
  + When replacing instances, it waits for the new instances to pass an initial health check\. It also waits for the default instance warmup to finish before continuing\.

**Note**  
If the size of an Auto Scaling group is small enough that the resulting value of 10 percent is less than one, Amazon EC2 Auto Scaling replaces the unhealthy instances one at a time instead\. This might result in some downtime for the group\.  
Also, if all instances in an Auto Scaling group are reported as unhealthy by Elastic Load Balancing health checks and the load balancer is in the `InService` state, Amazon EC2 Auto Scaling might mark fewer instances unhealthy at a time\. This can result in much fewer instances replaced at a time than the 10 percent applied in other scenarios\. This provides you time to address the problem without Amazon EC2 Auto Scaling automatically terminating the entire group\.

## Health check considerations<a name="health-check-considerations"></a>

This section contains considerations for Amazon EC2 Auto Scaling health checks\.
+ If you need something to happen on the instance that is terminating, or on the instance that is starting up, you can use lifecycle hooks\. These hooks let you perform a custom action as Amazon EC2 Auto Scaling launches or terminates instances\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\. 
+ To provide enough time for new instances to be ready to start accepting requests without being terminated due to failed health checks, do one, or both, of the following:
  + Add a lifecycle hook to the group\. This helps you make sure that your instances are fully configured before they are put into service at the end of the lifecycle hook\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\. 
  + Set the health check grace period of the group to match the expected startup period of your application\. For more information, see the [Health check grace period](#health-check-grace-period) section\. 
+ During the health check grace period, if Amazon EC2 Auto Scaling detects that an instance is no longer in the `running` state, it marks it as unhealthy and replaces it\. For example, this can happen when you stop an instance\. Amazon EC2 Auto Scaling has two options for stopping an `InService` instance\. You can either put the instance into standby, or you can detach it from the group\.
+ Amazon EC2 Auto Scaling does not provide a way of removing the Amazon EC2 status checks and scheduled events from its health checks\. If you do not want instances to be replaced, we recommend that you suspend the `ReplaceUnhealthy` and `HealthCheck` process for individual Auto Scaling groups\. For more information, see [Suspend and resume a process for an Auto Scaling group](as-suspend-resume-processes.md)\. 
+ To manually set an unhealthy instance's health status back to healthy, you can try to use the [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command\. If you get an error, this is probably because the instance is already terminating\. Generally, setting an instance's health status back to healthy with the [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command is only useful in cases where either the `ReplaceUnhealthy` process or the `Terminate` process is suspended\. 
+ Amazon EC2 Auto Scaling does not perform health checks on instances that are in the `Standby` state\. For more information, see [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md)\. 
+ When your instance is terminated, any associated Elastic IP addresses are disassociated and are not automatically associated with the new instance\. You must manually associate the Elastic IP addresses with the new instance, or do it automatically with a lifecycle hook\-based solution\. For more information, see [Elastic IP addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) in the *Amazon EC2 User Guide for Linux Instances*\.
+ Similarly, when your instance is terminated, its attached EBS volumes are detached \(or deleted depending on the volume's `DeleteOnTermination` attribute\)\. You must manually attach these EBS volumes to the new instance, or do it automatically with a lifecycle hook\-based solution\. For more information, see [Attach an Amazon EBS volume to an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html) in the *Amazon EC2 User Guide for Linux Instances*\.

## Health check grace period<a name="health-check-grace-period"></a>

The `HealthCheckGracePeriod` parameter for the Auto Scaling group helps Amazon EC2 Auto Scaling distinguish unhealthy instances from newly launched instances that are not yet ready to serve traffic\. This grace period can prevent Amazon EC2 Auto Scaling from marking `InService` instances as unhealthy and terminating them before they have time to finish initializing\.

By default, the health check grace period is `300` seconds when you create an Auto Scaling group from the AWS Management Console\. Its default value is `0` seconds when you create an Auto Scaling group using the AWS CLI or an SDK\. 

Set the health check grace period value greater than or equal to the maximum amount of startup time that your application needs\. Startup time begins when an instance starts and ends when it can receive traffic\. If you add a lifecycle hook, you can reduce the value of the health check grace period\. When lifecycle hooks are invoked, the grace period does not start until the lifecycle hook actions are completed and the instance enters the `InService` state\.

When you use the AWS CLI or an SDK to set the health status of instances as unhealthy, the default is to wait for the grace period to end\. However, you can override this behavior and not honor the grace period\. 

**To configure the health check grace period \(console\)**  
When you create the Auto Scaling group, on the **Configure advanced options** page, for **Health checks**, **Health check grace period**, choose the amount of startup time that your application needs\.

**To modify the health check grace period for an existing group \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling](https://console.aws.amazon.com/ec2autoscaling) and choose the AWS Region that you created your Auto Scaling group in\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. Under **Health check grace period**, choose the amount of startup time that your application needs\.

1. Choose **Update**\.

## Additional information<a name="health-checks-additional-information"></a>

For information about troubleshooting health checks, see [Troubleshoot Amazon EC2 Auto Scaling: Health checks](ts-as-healthchecks.md)\. If your health checks fail, check this topic for troubleshooting steps\. This topic will help you figure out what has gone wrong in your Auto Scaling group and give you suggestions on how to fix it\.

Amazon EC2 Auto Scaling also monitors the health of instances that you launch into a warm pool using Amazon EC2, Amazon EBS, or custom health checks\. For more information, see [View health check status and the reason for health check failures](warm-pools-health-checks-monitor-view-status.md)\.