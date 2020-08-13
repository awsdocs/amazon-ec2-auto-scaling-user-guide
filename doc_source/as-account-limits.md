# Amazon EC2 Auto Scaling service quotas<a name="as-account-limits"></a>

Your AWS account has the following default quotas, formerly referred to as limits, for Amazon EC2 Auto Scaling\. 

**Default quotas**
+ Launch configurations per Region: 200
+ Auto Scaling groups per Region: 200

To view the current quotas for your account, open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/) and navigate to the **Limits** page\. You can also use the [describe\-account\-limits](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-account-limits.html) command\. To request an increase, use the [Auto Scaling Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-auto-scaling)\.

**Auto Scaling group quotas**
+ Scaling policies per Auto Scaling group: 50
+ Scheduled actions per Auto Scaling group: 125
+ Lifecycle hooks per Auto Scaling group: 50
+ SNS topics per Auto Scaling group: 10
+ Classic Load Balancers per Auto Scaling group: 50
+ Target groups per Auto Scaling group: 50

**Scaling policy quotas**
+ Step adjustments per scaling policy: 20

**Auto Scaling API quotas**
+ You can use [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachInstances.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachInstances.html), [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html), [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnterStandby.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnterStandby.html), and [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExitStandby.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExitStandby.html) with at most 20 instance IDs at a time\.
+ You can use [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancers.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancers.html) and [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancers.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancers.html) with at most 10 load balancers at a time\.
+ You can use [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancerTargetGroups.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancerTargetGroups.html) and [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancerTargetGroups.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancerTargetGroups.html) with at most 10 target groups at a time\.

For information about the service quotas for other services, see [Service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/aws-service-information.html) in the *Amazon Web Services General Reference*\.