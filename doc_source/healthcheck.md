# Health Checks for Auto Scaling Instances<a name="healthcheck"></a>

The health status of an Auto Scaling instance is either healthy or unhealthy\. After an instance is fully configured and passes the initial health checks, it is considered healthy by Auto Scaling and enters the `InService` state\. Auto Scaling periodically performs health checks on the instances in your Auto Scaling group and identifies any instances that are unhealthy\. After Auto Scaling marks an instance as unhealthy, it is scheduled for replacement\. For more information, see [Replacing Unhealthy Instances](as-maintain-instance-levels.md#replace-unhealthy-instance)\.

## Instance Health Status<a name="instance-health-status"></a>

Auto Scaling determines the health status of an instance using one or more of the following:
+ Status checks provided by Amazon EC2 \(systems status checks and instance status checks\. For more information, see [Status Checks for Your Instances](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html) in the *Amazon EC2 User Guide for Linux Instances*\.
+ Health checks provided by Elastic Load Balancing\. For more information, see [Health Checks for Your Target Groups](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html) in the *User Guide for Application Load Balancers* or [Configure Health Checks for Your Classic Load Balancer](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-healthchecks.html) in the *User Guide for Classic Load Balancers*\.
+ Custom health checks\.

By default, Auto Scaling health checks use the results of the EC2 status checks to determine the health status of an instance\. Auto Scaling marks an instance as unhealthy if its instance fails one or more of the status checks\.

If you attached a load balancer or target group to your Auto Scaling group, you can configure Auto Scaling to mark an instance as unhealthy if Elastic Load Balancing reports the instance as `OutOfService`\. If connection draining is enabled for your load balancer, Auto Scaling waits for in\-flight requests to complete or the maximum timeout to expire, whichever comes first, before terminating instances due to a scaling event or health check replacement\. For more information, see [Using ELB Health Checks with Auto Scaling](as-add-elb-healthcheck.md)\.

## Health Check Grace Period<a name="health-check-grace-period"></a>

Frequently, an Auto Scaling instance that has just come into service needs to warm up before it can pass the Auto Scaling health check\. Auto Scaling waits until the health check grace period ends before checking the health status of the instance\. While the EC2 status checks and ELB health checks can complete before the health check grace period expires, Auto Scaling does not act on them until the health check grace period expires\. To provide ample warm\-up time for your instances, ensure that the health check grace period covers the expected startup time for your application\. Note that if you add a lifecycle hook to perform actions as your instances launch, the health check grace period does not start until the lifecycle hook is completed and the instance enters the `InService` state\.

## Custom Health Checks<a name="as-configure-healthcheck"></a>

If you have custom health checks, you can send the information from your health checks to Auto Scaling so that Auto Scaling can use this information\. For example, if you determine that an instance is not functioning as expected, you can set the health status of the instance to `Unhealthy`\. The next time that Auto Scaling performs a health check on the instance, it will determine that the instance is unhealthy and then launch a replacement instance\.

Use the following [set\-instance\-health](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-instance-health.html) command to set the health state of the specified instance to `Unhealthy`:

```
aws autoscaling set-instance-health --instance-id i-123abc45d --health-status Unhealthy
```

Use the following `describe-auto-scaling-groups` command to verify that the instance state is `Unhealthy`:

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names my-asg
```

The following is an example response that shows that the health status of the instance is `Unhealthy` and that the instance is terminating:

```
{
    "AutoScalingGroups": [
        {
            ....
            "Instances": [
                {
                    "InstanceId": "i-123abc45d",
                    "AvailabilityZone": "us-west-2a",
                    "HealthStatus": "Unhealthy",
                    "LifecycleState": "Terminating",
                    "LaunchConfigurationName": "my-lc"
                },
                ...
            ]
        }
    ]
}
```