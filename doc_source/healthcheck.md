# Health checks for Auto Scaling instances<a name="healthcheck"></a>

The health status of an Auto Scaling instance is either healthy or unhealthy\. All instances in your Auto Scaling group start in the healthy state\. Instances are assumed to be healthy unless Amazon EC2 Auto Scaling receives notification that they are unhealthy\. This notification can come from one or more of the following sources: Amazon EC2, Elastic Load Balancing, or a custom health check\. After Amazon EC2 Auto Scaling marks an instance as unhealthy, it is scheduled for replacement\.

**Topics**
+ [Instance health status](#instance-health-status)
+ [Determining instance health](#determine-instance-health)
+ [Considerations](#health-check-considerations)
+ [Health check grace period](#health-check-grace-period)
+ [Replacing unhealthy instances](#replace-unhealthy-instance)
+ [Using custom health checks](#as-configure-healthcheck)
+ [Additional information](#health-checks-additional-information)

## Instance health status<a name="instance-health-status"></a>

Amazon EC2 Auto Scaling can determine the health status of an instance using one or more of the following:
+ Status checks provided by Amazon EC2 to identify hardware and software issues that may impair an instance\. The default health checks for an Auto Scaling group are EC2 status checks only\.
+ Health checks provided by Elastic Load Balancing\. These health checks are disabled by default but can be enabled\.
+ Your custom health checks\. 

## Determining instance health<a name="determine-instance-health"></a>

After an instance is fully configured and passes the initial health checks, it is considered healthy by Amazon EC2 Auto Scaling and enters the `InService` state\.

Amazon EC2 Auto Scaling checks that all instances within the Auto Scaling group are running and in good shape by periodically checking the health state of the instances\. When it determines that an `InService` instance is unhealthy, it terminates that instance and launches a new one\. This helps in maintaining the number of running instances at the minimum number \(or desired number, if specified\) that you defined\.

**Amazon EC2 status checks**  
Amazon EC2 Auto Scaling health checks use the results of the Amazon EC2 status checks to determine the health status of an instance\. If the instance is in any *Amazon EC2 state* other than `running` or if the system status is `impaired`, Amazon EC2 Auto Scaling considers the instance to be unhealthy and replaces it\. This includes when the instance has any of the following states:
+ `stopping`
+ `stopped`
+ `shutting-down`
+ `terminated`

The EC2 status checks do not require any special configuration and are always enabled unless you suspend the `HealthCheck` process\. This includes both instance status checks and system status checks\. For more information, see [Types of status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html#types-of-instance-status-checks) in the *Amazon EC2 User Guide for Linux Instances*\. 

**Elastic Load Balancing \(`ELB`\) health checks**  
Instances for groups that do not use `ELB` health checks are considered healthy if they are in the `running` state\. Instances for groups that use `ELB` health checks are considered healthy if they are in the `running` state and they are reported as healthy by the load balancer\.

If you attached a load balancer or target group to your Auto Scaling group, you can configure the group to mark an instance as unhealthy when Elastic Load Balancing reports it as `unhealthy`\. If connection draining is enabled for your load balancer, Amazon EC2 Auto Scaling waits for in\-flight requests to complete or the maximum timeout to expire, whichever comes first, before terminating instances due to a scaling event or health check replacement\. 

For information about enabling these health checks, see [Adding ELB health checks](as-add-elb-healthcheck.md)\.

**Custom health checks**  
If you have custom health checks, you can send the information from your health checks to Amazon EC2 Auto Scaling so that it can use this information\. For example, if you determine that an instance is not functioning as expected, you can set the health status of the instance to `Unhealthy`\. The next time that Amazon EC2 Auto Scaling performs a health check on the instance, it will determine that the instance is unhealthy and then launch a replacement instance\. For more information, see [Using custom health checks](#as-configure-healthcheck)\. 

## Considerations<a name="health-check-considerations"></a>

To provide enough time for new instances to be ready to start serving application traffic without being terminated due to failed health checks, do one, or both, of the following:
+ Set the health check grace period of the group to match the expected startup period of your application\. For more information, see the [Health check grace period](#health-check-grace-period) section\. 
+ Add a lifecycle hook to the group to make sure that your instances are fully configured before they are ready to serve traffic at the end of the lifecycle hook\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

For Amazon EC2 status checks, consider the following:
+ When a system status check fails, Amazon EC2 Auto Scaling waits a few minutes for AWS to fix the issue\. It does not immediately mark instances as unhealthy when they have an `impaired` status\. However, any instances that are not in the `running` state are immediately marked as unhealthy\.
+ During the health check grace period, if an instance has already reached the `InService` state but it is no longer running, Amazon EC2 Auto Scaling marks it as unhealthy and replaces it\. For example, this can happen when you stop an instance\. Amazon EC2 Auto Scaling has two options for stopping an `InService` instance\. You can either put the instance into standby for a period of time, or you can detach it from the group\.
+ Amazon EC2 Auto Scaling does not provide a way of removing the Amazon EC2 status checks from its health checks\. If you do not want unhealthy instances to be replaced, we recommend that you suspend the `ReplaceUnhealthy` and `HealthCheck` process for individual Auto Scaling groups\. For more information, see [Suspending and resuming a process for an Auto Scaling group](as-suspend-resume-processes.md)\. 

## Health check grace period<a name="health-check-grace-period"></a>

The `HealthCheckGracePeriod` parameter for the Auto Scaling group helps Amazon EC2 Auto Scaling distinguish unhealthy instances from newly launched instances that are not yet ready to serve traffic\. This grace period can prevent Amazon EC2 Auto Scaling from marking instances as unhealthy and terminating them before they have time to come up\.

By default, the health check grace period is `300` seconds when you create an Auto Scaling group from the AWS Management Console\. Its default value is `0` seconds when you create an Auto Scaling group using the AWS CLI or an SDK\. 

Set this value greater than or equal to the maximum amount of startup time needed for your application, from when an instance starts to when it can receive traffic\. If you add a lifecycle hook, you can reduce the value of the health check grace period\. When there is a lifecycle hook, the grace period does not start until the lifecycle hook actions are completed and the instance enters the `InService` state\.

**To configure the health check grace period \(console\)**  
When you create the Auto Scaling group, on the **Configure advanced options** page, for **Health checks**, **Health check grace period**, choose the amount of startup time your application needs\.

**To modify the health check grace period for an existing group \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling](https://console.aws.amazon.com/ec2autoscaling) and choose the AWS Region that you created your Auto Scaling group in\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group you selected\. 

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. Under **Health check grace period**, choose the amount of startup time your application needs\.

1. Choose **Update**\.

## Replacing unhealthy instances<a name="replace-unhealthy-instance"></a>

After an instance has been marked unhealthy because of a health check, it is almost immediately scheduled for replacement\. It never automatically recovers its health\. You can intervene manually by calling the [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command to set the instance's health status back to healthy\. If the instance is already terminating, you get an error\. 

**Note**  
Because the interval between marking an instance unhealthy and its actual termination is so small, attempting to set an instance's health status back to healthy with the [set\-instance\-health](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command is probably useful only in cases where the `ReplaceUnhealthy` process is suspended\. For more information, see [Suspending and resuming a process for an Auto Scaling group](as-suspend-resume-processes.md)\.

Amazon EC2 Auto Scaling creates a new scaling activity for terminating the unhealthy instance and then terminates it\. Later, another scaling activity launches a new instance to replace the terminated instance\.

When your instance is terminated, any associated Elastic IP addresses are disassociated and are not automatically associated with the new instance\. You must associate these Elastic IP addresses with the new instance manually\. Similarly, when your instance is terminated, its attached EBS volumes are detached\. You must attach these EBS volumes to the new instance manually\. For more information, see [Disassociating an Elastic IP address and reassociating with a different instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating-different) and [Attaching an Amazon EBS volume to an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

## Using custom health checks<a name="as-configure-healthcheck"></a>

If you have your own health check system, you can send the instance's health information directly from your system to Amazon EC2 Auto Scaling using the AWS CLI or an SDK\. The following examples show how to use the AWS CLI to configure the health state of an instance and then verify the instance's health state\.

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

## Additional information<a name="health-checks-additional-information"></a>

For information about troubleshooting health checks, see [Troubleshooting Amazon EC2 Auto Scaling: Health checks](ts-as-healthchecks.md)\. If your health checks fail, check this topic for troubleshooting steps\. This topic will help you figure out what has gone wrong in your Auto Scaling group and give you suggestions on how to fix it\.

Amazon EC2 Auto Scaling also monitors the health of instances that you launch into a warm pool using Amazon EC2, Amazon EBS, or custom health checks\. For more information, see [Viewing health check status and the reason for health check failures](warm-pools-health-checks-monitor-view-status.md)\.