# Auto Scaling Limits<a name="as-account-limits"></a>

To view the current limits for your Auto Scaling resources, use the **Limits** page of the Amazon EC2 console or the [describe\-account\-limits](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-account-limits.html) \(AWS CLI\) command\. To request a limit increase, use the [Auto Scaling Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-auto-scaling)\.

The following limits are related to your Auto Scaling resources\.

**Regional Limits**

+ Launch configurations per region: 200

+ Auto Scaling groups per region: 200

**Auto Scaling Group Limits**

+ Scaling policies per Auto Scaling group: 50

+ Scheduled actions per Auto Scaling group: 125

+ Lifecycle hooks per Auto Scaling group: 50

+ SNS topics per Auto Scaling group: 10

+ Classic Load Balancers per Auto Scaling group: 50

+ Target groups per Auto Scaling group: 50

**Scaling Policy Limits**

+ Step adjustments per scaling policy: 20

**Auto Scaling API Limits**

+ You can use [AttachInstances](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachInstances.html), [DetachInstances](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html), [EnterStandby](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnterStandby.html), and [ExitStandby](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExitStandby.html) with at most 20 instance IDs at a time\.

+ You can use [AttachLoadBalancers](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancers.html) and [DetachLoadBalancers](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancers.html) with at most 10 load balancers at a time\.

+ You can use [AttachLoadBalancerTargetGroups](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancerTargetGroups.html) and [DetachLoadBalancerTargetGroups](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancerTargetGroups.html) with at most 10 target groups at a time\.