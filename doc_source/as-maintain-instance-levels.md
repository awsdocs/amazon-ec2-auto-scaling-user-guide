# Maintaining the Number of Instances in Your Auto Scaling Group<a name="as-maintain-instance-levels"></a>

After you have created your launch configuration and Auto Scaling group, the Auto Scaling group starts by launching the minimum number of EC2 instances \(or the desired capacity, if specified\)\. If there are no other scaling conditions attached to the Auto Scaling group, the Auto Scaling group maintains this number of running instances at all times\.

To maintain the same number of instances, Amazon EC2 Auto Scaling performs a periodic health check on running instances within an Auto Scaling group\. When it finds that an instance is unhealthy, it terminates that instance and launches a new one\.

All instances in your Auto Scaling group start in the healthy state\. Instances are assumed to be healthy unless Amazon EC2 Auto Scaling receives notification that they are unhealthy\. This notification can come from one or more of the following sources: Amazon EC2, Elastic Load Balancing, or your customized health check\.

## Determining Instance Health<a name="determine-instance-health"></a>

By default, the Auto Scaling group determines the health state of each instance by periodically checking the results of EC2 instance status checks\. If the instance status is any state other than `running` or if the system status is `impaired`, Amazon EC2 Auto Scaling considers the instance to be unhealthy and launches a replacement\. For more information about EC2 instance status checks, see [Monitoring the Status of Your Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-monitoring.html) in the *Amazon EC2 User Guide for Linux Instances*\.

If you have associated your Auto Scaling group with a load balancer or a target group and have chosen to use the Elastic Load Balancing health checks, Amazon EC2 Auto Scaling determines the health status of the instances by checking both the instance status checks and the Elastic Load Balancing health checks\. Amazon EC2 Auto Scaling marks an instance as unhealthy if the instance is in a state other than `running`, the system status is `impaired`, or Elastic Load Balancing reports that the instance failed the health checks\.

You can customize the health check conducted by your Auto Scaling group by specifying additional checks\. Or, if you have your own health check system, you can send the instance's health information directly from your system to Amazon EC2 Auto Scaling\.

## Replacing Unhealthy Instances<a name="replace-unhealthy-instance"></a>

After an instance has been marked unhealthy because of an Amazon EC2 or Elastic Load Balancing health check, it is almost immediately scheduled for replacement\. It never automatically recovers its health\. You can intervene manually by calling the [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceHealth.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceHealth.html) action \(or the as\-set\-instance\-health command\) to set the instance's health status back to healthy\. If the instance is already terminating, you get an error\. Because the interval between marking an instance unhealthy and its actual termination is so small, attempting to set an instance's health status back to healthy with the `SetInstanceHealth` action \(or, as\-set\-instance\-health command\) is probably useful only for a suspended group\. For more information, see [Suspending and Resuming Scaling Processes](as-suspend-resume-processes.md)\.

Amazon EC2 Auto Scaling creates a new scaling activity for terminating the unhealthy instance and then terminates it\. Later, another scaling activity launches a new instance to replace the terminated instance\.

When your instance is terminated, any associated Elastic IP addresses are disassociated and are not automatically associated with the new instance\. You must associate these Elastic IP addresses with the new instance manually\. Similarly, when your instance is terminated, its attached EBS volumes are detached\. You must attach these EBS volumes to the new instance manually\.