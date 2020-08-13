# Health checks for Auto Scaling instances<a name="healthcheck"></a>

The health status of an Auto Scaling instance is either healthy or unhealthy\. All instances in your Auto Scaling group start in the healthy state\. Instances are assumed to be healthy unless Amazon EC2 Auto Scaling receives notification that they are unhealthy\. This notification can come from one or more of the following sources: Amazon EC2, Elastic Load Balancing \(ELB\), or a custom health check\. 

After Amazon EC2 Auto Scaling marks an instance as unhealthy, it is scheduled for replacement\. If you do not want instances to be replaced, you can suspend the health check process for any individual Auto Scaling group\.

## Instance health status<a name="instance-health-status"></a>

Amazon EC2 Auto Scaling can determine the health status of an instance using one or more of the following:
+ Status checks provided by Amazon EC2 to identify hardware and software issues that may impair an instance\. The default health checks for an Auto Scaling group are EC2 status checks only\.
+ Health checks provided by Elastic Load Balancing \(ELB\)\. These health checks are disabled by default but can be enabled\.
+ Your custom health checks\. 

## Determining instance health<a name="determine-instance-health"></a>

After an instance is fully configured and passes the initial health checks, it is considered healthy by Amazon EC2 Auto Scaling\. Amazon EC2 Auto Scaling checks that all instances within the Auto Scaling group are running and in good shape by periodically checking the health state of the instances\. When it determines that an instance is unhealthy, it terminates that instance and launches a new one\. This helps in maintaining the number of running instances at the minimum number \(or desired number, if specified\) that you defined\.

**Amazon EC2 Status Checks**  
Amazon EC2 Auto Scaling health checks use the results of the Amazon EC2 status checks to determine the health status of an instance\. If the instance is in any state other than `running` or if the system status is `impaired`, Amazon EC2 Auto Scaling considers the instance to be unhealthy and launches a replacement instance\. This includes when the instance has any of the following states:
+ `stopping`
+ `stopped`
+ `terminating`
+ `terminated`

The EC2 status checks do not require any special configuration and are always enabled\. This includes both instance status checks and system status checks\. For more information, see [Types of status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html#types-of-instance-status-checks) in the *Amazon EC2 User Guide for Linux Instances*\. 

**Elastic Load Balancing \(`ELB`\) Health Checks**  
Instances for groups that do not use `ELB` health checks are considered healthy if they are in the `running` state\. Instances for groups that use `ELB` health checks are considered healthy if they are in the `running` state and they are reported as healthy by the load balancer\.

If you attached a load balancer or target group to your Auto Scaling group, you can configure the group to mark an instance as unhealthy when Elastic Load Balancing reports it as `unhealthy`\. If connection draining is enabled for your load balancer, Amazon EC2 Auto Scaling waits for in\-flight requests to complete or the maximum timeout to expire, whichever comes first, before terminating instances due to a scaling event or health check replacement\. For more information, see [Adding ELB health checks](as-add-elb-healthcheck.md)\.

**Custom Health Checks**  
If you have custom health checks, you can send the information from your health checks to Amazon EC2 Auto Scaling so that Amazon EC2 Auto Scaling can use this information\. For example, if you determine that an instance is not functioning as expected, you can set the health status of the instance to `Unhealthy`\. The next time that Amazon EC2 Auto Scaling performs a health check on the instance, it will determine that the instance is unhealthy and then launch a replacement instance\. For more information, see [Using custom health checks](#as-configure-healthcheck)\. 

## Health check grace period<a name="health-check-grace-period"></a>

When an instance launches, Amazon EC2 Auto Scaling uses the value of the `HealthCheckGracePeriod` for the Auto Scaling group to determine how long to wait before checking the health status of the instance\. Amazon EC2 and Elastic Load Balancing health checks can complete before the health check grace period expires\. However, Amazon EC2 Auto Scaling does not act on them until the health check grace period expires\. 

By default, the health check grace period is `300` seconds when you create an Auto Scaling group from the AWS Management Console\. Its default value is `0` seconds when you create an Auto Scaling group using the AWS CLI or an AWS SDK\. 

To provide ample warm\-up time for your instances, ensure that the health check grace period covers the expected startup time for your application, from when an instance comes into service to when it can receive traffic\. If you add a lifecycle hook, the grace period does not start until the lifecycle hook actions are completed and the instance enters the `InService` state\.

## Replacing unhealthy instances<a name="replace-unhealthy-instance"></a>

After an instance has been marked unhealthy because of a health check, it is almost immediately scheduled for replacement\. It never automatically recovers its health\. You can intervene manually by calling the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command or the [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceHealth.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceHealth.html) operation to set the instance's health status back to healthy\. If the instance is already terminating, you get an error\. 

**Note**  
Because the interval between marking an instance unhealthy and its actual termination is so small, attempting to set an instance's health status back to healthy with the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command or the [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceHealth.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceHealth.html) operation is probably useful only for a suspended group\. For more information, see [Suspending and resuming scaling processes](as-suspend-resume-processes.md)\.

Amazon EC2 Auto Scaling creates a new scaling activity for terminating the unhealthy instance and then terminates it\. Later, another scaling activity launches a new instance to replace the terminated instance\.

When your instance is terminated, any associated Elastic IP addresses are disassociated and are not automatically associated with the new instance\. You must associate these Elastic IP addresses with the new instance manually\. Similarly, when your instance is terminated, its attached EBS volumes are detached\. You must attach these EBS volumes to the new instance manually\. For more information, see [Disassociating an Elastic IP address and reassociating with a different instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating-different) and [Attaching an Amazon EBS volume to an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

## Using custom health checks<a name="as-configure-healthcheck"></a>

If you have your own health check system, you can send the instance's health information directly from your system to Amazon EC2 Auto Scaling using the AWS CLI or an AWS SDK\. The following examples show how to use the AWS CLI to configure the health state of an instance and then verify the instance's health state\.

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command to set the health state of the specified instance to `Unhealthy`\.

```
aws autoscaling set-instance-health --instance-id i-123abc45d --health-status Unhealthy
```

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the instance state is `Unhealthy`\.

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