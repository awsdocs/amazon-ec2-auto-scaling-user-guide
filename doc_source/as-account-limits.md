# Amazon EC2 Auto Scaling Limits<a name="as-account-limits"></a>

Your AWS account has the following limits related to Amazon EC2 Auto Scaling\. For information about the service limits for other services, see [AWS Service Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html) in the *Amazon Web Services General Reference*\.

**Default Limits**
+ Launch configurations per Region: 200
+ Auto Scaling groups per Region: 200

To view the current limits for your account, use the **Limits** page of the Amazon EC2 console or the [describe\-account\-limits](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-account-limits.html) \(AWS CLI\) command\. To request a limit increase, use the [Auto Scaling Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-auto-scaling)\.

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
+ You can use [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachInstances.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachInstances.html), [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html), [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnterStandby.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnterStandby.html), and [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExitStandby.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExitStandby.html) with at most 20 instance IDs at a time\.
+ You can use [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancers.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancers.html) and [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancers.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancers.html) with at most 10 load balancers at a time\.
+ You can use [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancerTargetGroups.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancerTargetGroups.html) and [https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancerTargetGroups.html](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancerTargetGroups.html) with at most 10 target groups at a time\.