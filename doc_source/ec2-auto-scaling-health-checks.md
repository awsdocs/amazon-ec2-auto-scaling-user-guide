# Health checks for Auto Scaling instances<a name="ec2-auto-scaling-health-checks"></a>

The health status of an Auto Scaling instance is either healthy or unhealthy\. All instances in your Auto Scaling group start in the healthy state\. Instances are assumed to be healthy unless Amazon EC2 Auto Scaling receives notification that they are unhealthy\. This notification can come from one or more of the following sources: Amazon EC2, Elastic Load Balancing, or a custom health check\. After Amazon EC2 Auto Scaling detects that an instance is unhealthy, it terminates that instance and launches a new one\. 

**Topics**
+ [Available health checks](#available-health-checks)
+ [Instance health detection](#instance-health-detection)
+ [Elastic Load Balancing health checks](#elastic-load-balancing-health-checks)
+ [Custom health detection tasks](#as-configure-healthcheck)
+ [Replace unhealthy instances](#replace-unhealthy-instance)
+ [Health check considerations](#health-check-considerations)
+ [Health check grace period](#health-check-grace-period)
+ [Additional information](#health-checks-additional-information)

## Available health checks<a name="available-health-checks"></a>

Amazon EC2 Auto Scaling can determine the health status of an instance using one or more of the following:


****  

| Health check type | What does it check? | 
| --- | --- | 
| Amazon EC2 status checks and scheduled events |  Ensures the instance is running and that there are no underlying hardware or software issues that may impair the instance\. The default health checks for an Auto Scaling group are Amazon EC2 status checks and scheduled events only\.   | 
| Elastic Load Balancing health checks | Checks whether the instance is reported as healthy by the load balancer to ensure that the instance is available to handle requests\. Only checked if you have enabled these health checks for an Auto Scaling group\. | 
| Custom health checks | Ensures that there are no other problems that indicate instance health issues as per your custom health checks\. | 

## Instance health detection<a name="instance-health-detection"></a>

After an instance launches, it is attached to the Auto Scaling group and enters the `InService` state\. For more information about the different lifecycle states for instances in an Auto Scaling group, see [Amazon EC2 Auto Scaling instance lifecycle](ec2-auto-scaling-lifecycle.md)\.

Amazon EC2 Auto Scaling checks that all instances within the Auto Scaling group are running and in good shape by periodically checking the health status of the instances\.

**Status checks**  
Amazon EC2 Auto Scaling uses the results of the Amazon EC2 instance status checks and system status checks to determine the health status of an instance\. If the instance is in any Amazon EC2 state other than `running` or its status for the status checks becomes `impaired`, Amazon EC2 Auto Scaling considers the instance to be unhealthy and replaces it\. This includes when the instance has any of the following states:
+ `stopping`
+ `stopped`
+ `shutting-down`
+ `terminated`

The Amazon EC2 status checks do not require any special configuration and are always enabled\. For more information, see [Types of status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html#types-of-instance-status-checks) in the *Amazon EC2 User Guide for Linux Instances*\. 

**Important**  
Amazon EC2 Auto Scaling allows these status checks to occasionally fail without taking any action\. When a status check fails, Amazon EC2 Auto Scaling waits a few minutes for AWS to fix the issue\. It does not immediately mark an instance as unhealthy when its status for the status checks becomes `impaired`\.  
If Amazon EC2 Auto Scaling detects that an instance is no longer in the `running` state, however, this situation is treated as an immediate failure, in which case it immediately marks the instance as unhealthy and replaces it\. 

**Scheduled events**  
A scheduled event is a future scheduled reboot or retirement event\. If one of your instances is affected by a scheduled event, Amazon EC2 Auto Scaling considers the instance to be unhealthy and replaces it almost immediately\. It does not wait for the scheduled time to be reached\. For more information, see [Types of scheduled events](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-instances-status-check_sched.html#types-of-scheduled-events) in the *Amazon EC2 User Guide for Linux Instances*\.

## Elastic Load Balancing health checks<a name="elastic-load-balancing-health-checks"></a>

Amazon EC2 Auto Scaling can also use the results of the Elastic Load Balancing health checks to determine the health status of an instance\. When you attach an Elastic Load Balancing load balancer to your Auto Scaling group, Amazon EC2 Auto Scaling registers the instances with the load balancer\. After an instance finishes registering, it enters the `InService` state and becomes available for use with the load balancer\. 

By default, the Elastic Load Balancing health checks are not enabled\. When you enable these health checks and Elastic Load Balancing reports a registered instance as `unhealthy`, Amazon EC2 Auto Scaling marks the instance as unhealthy on its next periodic health check and replaces it\. If connection draining \(deregistration delay\) is enabled for your load balancer, Amazon EC2 Auto Scaling waits for in\-flight requests to complete or the maximum timeout to expire, whichever comes first, before terminating unhealthy instances\. 

For more information, including how to update your Auto Scaling group to enable Elastic Load Balancing health checks, see [Add Elastic Load Balancing health checks to an Auto Scaling group](as-add-elb-healthcheck.md)\.

## Custom health detection tasks<a name="as-configure-healthcheck"></a>

Sometimes you also want to run custom health detection tasks on the instances in your Auto Scaling group and set the health status of an instance as unhealthy if the task fails\. By doing so, your health checks are extended by using a combination of custom health checks, Amazon EC2 status checks, and Elastic Load Balancing health checks, if enabled\.

You can send the instance's health information directly to Amazon EC2 Auto Scaling using the AWS CLI or an SDK\. The following examples show how to use the AWS CLI to configure the health state of an instance and then verify the instance's health state\.

Use the following [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command to set the health state of the specified instance to `Unhealthy`\.

```
aws autoscaling set-instance-health --instance-id i-123abc45d --health-status Unhealthy
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
                        "LaunchTemplateId": "lt-050555ad16a3f9c7f"
                    },
                    "InstanceId": "i-123abc45d",
                    "HealthStatus": "Unhealthy",
                    "LifecycleState": "Terminating"
                },
                ...
            ]
        }
    ]
}
```

## Replace unhealthy instances<a name="replace-unhealthy-instance"></a>

When Amazon EC2 Auto Scaling determines that an `InService` instance is unhealthy, the instance is almost immediately terminated\. It never automatically recovers its health\. You can intervene manually by calling the [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command to set the instance's health status back to healthy\. If the instance is already terminating, you get an error\. 

Because the interval between marking an instance unhealthy and its actual termination is so small, attempting to set an instance's health status back to healthy with the [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command is probably useful only in cases where the `ReplaceUnhealthy` process is suspended\. For more information, see [Suspend and resume a process for an Auto Scaling group](as-suspend-resume-processes.md)\.

Amazon EC2 Auto Scaling creates a new scaling activity for terminating the unhealthy instance and then terminates it\. Then, another scaling activity launches a new instance to replace the terminated instance\.

When you view the **Activity** tab for your Auto Scaling group, you will see these activities listed in the activity history\. For more information, see [Verify a scaling activity for an Auto Scaling group](as-verify-scaling-activity.md)\.

If you need something to happen on the instance that is terminating, or on the instance that is starting up, you can use lifecycle hooks\. These hooks allow you to perform a custom action as Amazon EC2 Auto Scaling launches or terminates instances\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\. 

## Health check considerations<a name="health-check-considerations"></a>
+ To provide enough time for new instances to be ready to start handling requests without being terminated due to failed health checks, do one, or both, of the following:
  + Set the health check grace period of the group to match the expected startup period of your application\. For more information, see the [Health check grace period](#health-check-grace-period) section\. 
  + Add a lifecycle hook to the group to make sure that your instances are fully configured before they are ready to serve traffic at the end of the lifecycle hook\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.
+ During the health check grace period, if Amazon EC2 Auto Scaling detects that an instance is no longer in the `running` state, it marks it as unhealthy and replaces it\. For example, this can happen when you stop an instance\. Amazon EC2 Auto Scaling has two options for stopping an `InService` instance\. You can either put the instance into standby for a period of time, or you can detach it from the group\.
+ Amazon EC2 Auto Scaling does not provide a way of removing the Amazon EC2 status checks and scheduled events from its health checks\. If you do not want instances to be replaced, we recommend that you suspend the `ReplaceUnhealthy` and `HealthCheck` process for individual Auto Scaling groups\. For more information, see [Suspend and resume a process for an Auto Scaling group](as-suspend-resume-processes.md)\. 
+ Amazon EC2 Auto Scaling does not perform health checks on instances that are in the `Standby` state\. For more information, see [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md)\. 
+ When your instance is terminated, any associated Elastic IP addresses are disassociated and are not automatically associated with the new instance\. You must associate these Elastic IP addresses with the new instance manually, or automatically with a lifecycle hook\-based solution\. For more information, see [Elastic IP addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) in the *Amazon EC2 User Guide for Linux Instances*
+ Similarly, when your instance is terminated, its attached EBS volumes are detached \(or deleted depending on the volume's `DeleteOnTermination` attribute\)\. You must attach these EBS volumes to the new instance manually, or automatically with a lifecycle hook\-based solution\. For more information, see [Attach an Amazon EBS volume to an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html) in the *Amazon EC2 User Guide for Linux Instances*

## Health check grace period<a name="health-check-grace-period"></a>

The `HealthCheckGracePeriod` parameter for the Auto Scaling group helps Amazon EC2 Auto Scaling distinguish unhealthy instances from newly launched instances that are not yet ready to serve traffic\. This grace period can prevent Amazon EC2 Auto Scaling from marking `InService` instances as unhealthy and terminating them before they have time to finish initializing\.

By default, the health check grace period is `300` seconds when you create an Auto Scaling group from the AWS Management Console\. Its default value is `0` seconds when you create an Auto Scaling group using the AWS CLI or an SDK\. 

Set this value greater than or equal to the maximum amount of startup time needed for your application, from when an instance starts to when it can receive traffic\. If you add a lifecycle hook, you can reduce the value of the health check grace period\. When lifecycle hooks are invoked, the grace period does not start until the lifecycle hook actions are completed and the instance enters the `InService` state\.

When you use the AWS CLI or an SDK to set the health status of instances as unhealthy, the default is to wait for the grace period to end\. However, you can override this behavior and not honor the grace period\. 

**To configure the health check grace period \(console\)**  
When you create the Auto Scaling group, on the **Configure advanced options** page, for **Health checks**, **Health check grace period**, choose the amount of startup time your application needs\.

**To modify the health check grace period for an existing group \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling](https://console.aws.amazon.com/ec2autoscaling) and choose the AWS Region that you created your Auto Scaling group in\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. Under **Health check grace period**, choose the amount of startup time your application needs\.

1. Choose **Update**\.

## Additional information<a name="health-checks-additional-information"></a>

For information about troubleshooting health checks, see [Troubleshoot Amazon EC2 Auto Scaling: Health checks](ts-as-healthchecks.md)\. If your health checks fail, check this topic for troubleshooting steps\. This topic will help you figure out what has gone wrong in your Auto Scaling group and give you suggestions on how to fix it\.

Amazon EC2 Auto Scaling also monitors the health of instances that you launch into a warm pool using Amazon EC2, Amazon EBS, or custom health checks\. For more information, see [View health check status and the reason for health check failures](warm-pools-health-checks-monitor-view-status.md)\.