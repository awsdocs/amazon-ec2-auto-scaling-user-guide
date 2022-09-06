# Examples for working with Elastic Load Balancing with the AWS Command Line Interface \(AWS CLI\)<a name="examples-elastic-load-balancing-aws-cli"></a>

Use the AWS CLI to attach and detach load balancers, add Elastic Load Balancing health checks, and update Availability Zones\.

**Topics**
+ [Attach a load balancer target group](#example-attach-load-balancer-target-group)
+ [Describe load balancer target groups](#example-describe-load-balancer-target-groups)
+ [Detach a load balancer target group](#example-detach-load-balancer-target-group)
+ [Attach a Classic Load Balancer](#example-attach-classic-load-balancer)
+ [Describe Classic Load Balancers](#example-describe-load-balancers)
+ [Detach a Classic Load Balancer](#example-detach-classic-load-balancer)
+ [Add Elastic Load Balancing health checks](#example-add-elb-healthcheck)
+ [Update Availability Zones](#example-specify-availability-zones)

## Attach a load balancer target group<a name="example-attach-load-balancer-target-group"></a>

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group with an attached target group\. Specify the Amazon Resource Name \(ARN\) of a target group for an Application Load Balancer, Network Load Balancer, or Gateway Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template "LaunchTemplateName=my-launch-template,Version=1" \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --target-group-arns "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/1234567890123456" \
  --max-size 5 --min-size 1 --desired-capacity 2
```

The following [attach\-load\-balancer\-target\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancer-target-groups.html) command attaches a target group to an existing Auto Scaling group\.

```
aws autoscaling attach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/1234567890123456"
```

## Describe load balancer target groups<a name="example-describe-load-balancer-target-groups"></a>

To view the target groups associated with an Auto Scaling group, use the [describe\-load\-balancer\-target\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-load-balancer-target-groups.html) command\. The following example lists the target groups for *my\-asg*\. 

```
aws autoscaling describe-load-balancer-target-groups --auto-scaling-group-name my-asg
```

For an explanation of the `State` field in the output, see the [Understand the attachment status of your load balancer](load-balancer-status.md) section in a previous topic\.

## Detach a load balancer target group<a name="example-detach-load-balancer-target-group"></a>

The following [detach\-load\-balancer\-target\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancer-target-groups.html) command detaches a target group from your Auto Scaling group when you no longer need it\. 

```
aws autoscaling detach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/1234567890123456"
```

## Attach a Classic Load Balancer<a name="example-attach-classic-load-balancer"></a>

The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group with an attached Classic Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --load-balancer-names "my-load-balancer" \
  --max-size 5 --min-size 1 --desired-capacity 2
```

The following [attach\-load\-balancers](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancers.html) command attaches the specified Classic Load Balancer to an existing Auto Scaling group\.

```
aws autoscaling attach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```

## Describe Classic Load Balancers<a name="example-describe-load-balancers"></a>

To view the Classic Load Balancers associated with an Auto Scaling group, use the [describe\-load\-balancers](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-load-balancers.html) command\. The following example lists the Classic Load Balancers for *my\-asg*\. 

```
aws autoscaling describe-load-balancers --auto-scaling-group-name my-asg
```

For an explanation of the `State` field in the output, see [Understand the attachment status of your load balancer](load-balancer-status.md)\.

## Detach a Classic Load Balancer<a name="example-detach-classic-load-balancer"></a>

The following [detach\-load\-balancers](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancers.html) command detaches a Classic Load Balancer from your Auto Scaling group when you no longer need it\.

```
aws autoscaling detach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```

## Add Elastic Load Balancing health checks<a name="example-add-elb-healthcheck"></a>

To add Elastic Load Balancing health checks to an Auto Scaling group, run the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command and specify `ELB` as the value for the `--health-check-type` option\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-lb-asg \
  --health-check-type ELB
```

To update the health check grace period, use the `--health-check-grace-period` option\. New instances often need time for a brief warm\-up before they can pass a health check\. If the grace period doesn't provide enough warm\-up time, the instances might not appear ready to serve traffic\. Amazon EC2 Auto Scaling might consider those instances unhealthy and replace them\. For more information, see [Health check grace period](ec2-auto-scaling-health-checks.md#health-check-grace-period)\.

The following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command adds Elastic Load Balancing health checks and specifies a grace period of 300 seconds\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-lb-asg \
  --health-check-type ELB --health-check-grace-period 300
```

## Update Availability Zones<a name="example-specify-availability-zones"></a>

The commands that you use depend on whether your load balancer is an Application Load Balancer or Network Load Balancer, or a Classic Load Balancer\. You can update the subnets and Availability Zones for your load balancer only if your load balancer supports it\. For more information, see [Limitations](as-add-availability-zone.md#availability-zone-limitations)\.

**For an Auto Scaling group with an Application Load Balancer or Network Load Balancer**

1. Specify the subnets that are used for the Auto Scaling group using the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --vpc-zone-identifier subnet-41767929 subnet-cb663da2 subnet-8360a9e7
   ```

1. Verify that the instances in the new subnets are ready to accept traffic from the load balancer using the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Specify the subnets that are used for your Application Load Balancer or Network Load Balancer using the following [set\-subnets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-subnets.html) command\.

   ```
   aws elbv2 set-subnets --load-balancer-arn my-lb-arn \
     --subnets subnet-41767929 subnet-cb663da2 subnet-8360a9e7
   ```

**For an Auto Scaling group with a Classic Load Balancer in a VPC**

1. Specify the subnets that are used for the Auto Scaling group using the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --vpc-zone-identifier subnet-41767929 subnet-cb663da2
   ```

1. Verify that the instances in the new subnets are ready to accept traffic from the load balancer using the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Enable the new subnet for your Classic Load Balancer using the following [attach\-load\-balancer\-to\-subnets](https://docs.aws.amazon.com/cli/latest/reference/elb/attach-load-balancer-to-subnets.html) command\.

   ```
   aws elb attach-load-balancer-to-subnets --load-balancer-name my-lb \
     --subnets subnet-cb663da2
   ```

   To disable a subnet, run the following [detach\-load\-balancer\-from\-subnets](https://docs.aws.amazon.com/cli/latest/reference/elb/detach-load-balancer-from-subnets.html) command\.

   ```
   aws elb detach-load-balancer-from-subnets --load-balancer-name my-lb \
     --subnets subnet-8360a9e7
   ```