# Examples for working with Elastic Load Balancing with the AWS Command Line Interface \(AWS CLI\)<a name="examples-elastic-load-balancing-aws-cli"></a>

Use the AWS CLI to attach, detach, and describe load balancers and target groups, add and remove Elastic Load Balancing health checks, and change which Availability Zones are enabled\.

This topic shows examples of AWS CLI commands that perform common tasks for Amazon EC2 Auto Scaling\.

**Important**  
For additional command examples, see [https://docs.aws.amazon.com/cli/latest/reference/elbv2/index.html](https://docs.aws.amazon.com/cli/latest/reference/elbv2/index.html) and [https://docs.aws.amazon.com/cli/latest/reference/elb/index.html](https://docs.aws.amazon.com/cli/latest/reference/elb/index.html) in the *AWS CLI Command Reference*\.

**Topics**
+ [Attach your target group or Classic Load Balancer](#example-attach-traffic-sources)
+ [Describe your target groups or Classic Load Balancers](#example-describe-traffic-sources)
+ [Add Elastic Load Balancing health checks](#example-add-elb-healthcheck)
+ [Change your Availability Zones](#example-specify-availability-zones)
+ [Detach your target group or Classic Load Balancer](#example-detach-traffic-sources)
+ [Remove Elastic Load Balancing health checks](#example-remove-elb-healthcheck)
+ [Legacy commands](#legacy-commands)

## Attach your target group or Classic Load Balancer<a name="example-attach-traffic-sources"></a>

Use the following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group and simultaneously attach a target group by specifying its Amazon Resource Name \(ARN\)\. The target group can be associated with an Application Load Balancer, a Network Load Balancer, or a Gateway Load Balancer\. 

Replace the sample values for `--auto-scaling-group-name`, `--vpc-zone-identifier`, `--min-size`, and `--max-size`\. For the `--launch-template` option, replace `my-launch-template` and `1` with the name and version of a launch template for your Auto Scaling group\. For the `--traffic-sources` option, replace the sample ARN with the ARN of a target group for an Application Load Balancer, Network Load Balancer, or Gateway Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-launch-template,Version='1' \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --min-size 1 --max-size 5 \
  --traffic-sources "Identifier=arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/12345678EXAMPLE1"
```

Use the [attach\-traffic\-sources](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-traffic-sources.html) command to attach additional target groups to the Auto Scaling group after it's created\.

The following command adds another target group to the same group\.

```
aws autoscaling attach-traffic-sources --auto-scaling-group-name my-asg \
  --traffic-sources "Identifier=arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/12345678EXAMPLE2"
```

Alternatively, to attach a Classic Load Balancer to your group, specify the `--traffic-sources` and `--type` options when you use create\-auto\-scaling\-group or attach\-traffic\-sources, as in the following example\. Replace `my-classic-load-balancer` with the name of a Classic Load Balancer\. For the `--type` option, specify a value of `elb`\.

```
--traffic-sources "Identifier=my-classic-load-balancer" --type elb
```

## Describe your target groups or Classic Load Balancers<a name="example-describe-traffic-sources"></a>

To describe the load balancers or target groups attached to your Auto Scaling group, use the following [describe\-traffic\-sources](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-traffic-sources.html) command\. Replace `my-asg` with the name of your group\. 

```
aws autoscaling describe-traffic-sources --auto-scaling-group-name my-asg
```

The example returns the ARN of the Elastic Load Balancing target groups that you attached to the Auto Scaling group\.

```
{
    "TrafficSources": [
        {
            "Identifier": "arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/12345678EXAMPLE1",
            "State": "InService",
            "Type": "elbv2"
        },
        {
            "Identifier": "arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/12345678EXAMPLE2",
            "State": "InService",
            "Type": "elbv2"
        }
    ]
}
```

For an explanation of the `State` field in the output, see [Verify the attachment status of your load balancer](load-balancer-status.md)\.

## Add Elastic Load Balancing health checks<a name="example-add-elb-healthcheck"></a>

To add Elastic Load Balancing health checks to the health checks that your Auto Scaling group performs on instances, use the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command and specify `ELB` as the value for the `--health-check-type` option\. Replace `my-asg` with the name of your group\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --health-check-type "ELB"
```

New instances often need time for a brief warm\-up before they can pass a health check\. If the grace period doesn't provide enough warm\-up time, the instances might not appear ready to serve traffic\. Amazon EC2 Auto Scaling might consider those instances unhealthy and replace them\.

To update the health check grace period, use the `--health-check-grace-period` option when you use update\-auto\-scaling\-group, as in the following example\. Replace *300* with the number of seconds to keep new instances in service before terminating them if they're found to be unhealthy\.

```
--health-check-grace-period 300
```

For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\.

## Change your Availability Zones<a name="example-specify-availability-zones"></a>

Changing your Availability Zones has some limitations that you should be aware of\. For more information, see [Limitations](as-add-availability-zone.md#availability-zone-limitations)\.

**To change the Availability Zones for an Application Load Balancer or Network Load Balancer**

1. Before you change the Availability Zones of the load balancer, it's a good idea to first update the Availability Zones of the Auto Scaling group to verify that there is availability for your instance types in the specified zones\. 

   To update the Availability Zones for your Auto Scaling group, use the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\. Replace the sample subnet IDs with the IDs of the subnets in the Availability Zones to enable\. The specified subnets replace the previously enabled subnets\. Replace `my-asg` with the name of your group\. 

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --vpc-zone-identifier "subnet-41767929,subnet-cb663da2,subnet-8360a9e7"
   ```

1. Use the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the instances in the new subnets have launched\. If the instances have launched, you see a list of the instances and their statuses\. Replace `my-asg` with the name of your group\. 

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Use the following [set\-subnets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-subnets.html) command to specify the subnets for your load balancer\. Replace the sample subnet IDs with the IDs of the subnets in the Availability Zones to enable\. You can specify only one subnet per Availability Zone\. The specified subnets replace the previously enabled subnets\. Replace `my-lb-arn` with the ARN of your load balancer\. 

   ```
   aws elbv2 set-subnets --load-balancer-arn my-lb-arn \
     --subnets subnet-41767929 subnet-cb663da2 subnet-8360a9e7
   ```

**To change the Availability Zones for a Classic Load Balancer**

1. Before you change the Availability Zones of the load balancer, it's a good idea to first update the Availability Zones of the Auto Scaling group to verify that there is availability for your instance types in the specified zones\. 

   To update the Availability Zones for your Auto Scaling group, use the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\. Replace the sample subnet IDs with the IDs of the subnets in the Availability Zones to enable\. The specified subnets replace the previously enabled subnets\. Replace `my-asg` with the name of your group\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --vpc-zone-identifier "subnet-41767929,subnet-cb663da2"
   ```

1. Use the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the instances in the new subnets have launched\. If the instances have launched, you see a list of the instances and their statuses\. Replace `my-asg` with the name of your group\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Use the following [attach\-load\-balancer\-to\-subnets](https://docs.aws.amazon.com/cli/latest/reference/elb/attach-load-balancer-to-subnets.html) command to enable a new Availability Zone for your Classic Load Balancer\. Replace the sample subnet ID with the ID of the subnet for the Availability Zone to enable\. Replace `my-lb` with the name of your load balancer\. 

   ```
   aws elb attach-load-balancer-to-subnets --load-balancer-name my-lb \
     --subnets subnet-cb663da2
   ```

   To disable an Availability Zone, use the following [detach\-load\-balancer\-from\-subnets](https://docs.aws.amazon.com/cli/latest/reference/elb/detach-load-balancer-from-subnets.html) command\. Replace the sample subnet ID with the ID of the subnet for the Availability Zone to disable\. Replace `my-lb` with the name of your load balancer\. 

   ```
   aws elb detach-load-balancer-from-subnets --load-balancer-name my-lb \
     --subnets subnet-8360a9e7
   ```

## Detach your target group or Classic Load Balancer<a name="example-detach-traffic-sources"></a>

The following [detach\-traffic\-sources](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-traffic-sources.html) command detaches a target group from your Auto Scaling group when you no longer need it\. 

For the `--auto-scaling-group-name` option, replace `my-asg` with the name of your group\. For the `--traffic-sources` option, replace the sample ARN with the ARN of a target group for an Application Load Balancer, Network Load Balancer, or Gateway Load Balancer\.

```
aws autoscaling detach-traffic-sources --auto-scaling-group-name my-asg \
  --traffic-sources "Identifier=arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/1234567890123456"
```

To detach a Classic Load Balancer from your group, specify the `--traffic-sources` and `--type` options, as in the following example\. Replace `my-classic-load-balancer` with the name of a Classic Load Balancer\. For the `--type` option, specify a value of `elb`\.

```
--traffic-sources "Identifier=my-classic-load-balancer" --type elb
```

## Remove Elastic Load Balancing health checks<a name="example-remove-elb-healthcheck"></a>

To remove Elastic Load Balancing health checks from your Auto Scaling group, use the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command and specify `EC2` as the value for the `--health-check-type` option\. Replace `my-asg` with the name of your group\. 

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --health-check-type "EC2"
```

For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\.

## Legacy commands<a name="legacy-commands"></a>

The following examples show how you can use legacy CLI commands to attach, detach, and describe load balancers and target groups\. They remain in this document as a reference for any customers who want to use them\. We continue to support the legacy CLI commands, but we recommend that you use the new "traffic sources" CLI commands, which can attach and detach multiple traffic sources types\. You can use both the legacy CLI commands and the "traffic sources" CLI commands on the same Auto Scaling group\.

### Attach your target group or Classic Load Balancer \(legacy\)<a name="example-attach-load-balancer-target-group"></a>

**To attach your target group**  
The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group with an attached target group\. Specify the Amazon Resource Name \(ARN\) of a target group for an Application Load Balancer, Network Load Balancer, or Gateway Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-launch-template,Version='1' \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --target-group-arns "arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/1234567890123456" \
  --min-size 1 --max-size 5
```

The following [attach\-load\-balancer\-target\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancer-target-groups.html) command attaches a target group to an existing Auto Scaling group\.

```
aws autoscaling attach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns "arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/1234567890123456"
```

**To attach your Classic Load Balancer**  
The following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command creates an Auto Scaling group with an attached Classic Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --load-balancer-names "my-load-balancer" \
  --min-size 1 --max-size 5
```

The following [attach\-load\-balancers](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancers.html) command attaches the specified Classic Load Balancer to an existing Auto Scaling group\.

```
aws autoscaling attach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```

### Describe your target group or Classic Load Balancer \(legacy\)<a name="example-describe-load-balancer-target-groups"></a>

**To describe target groups**  
To describe the target groups associated with an Auto Scaling group, use the [describe\-load\-balancer\-target\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-load-balancer-target-groups.html) command\. The following example lists the target groups for *my\-asg*\. 

```
aws autoscaling describe-load-balancer-target-groups --auto-scaling-group-name my-asg
```

**To describe Classic Load Balancers**  
To describe the Classic Load Balancers associated with an Auto Scaling group, use the [describe\-load\-balancers](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-load-balancers.html) command\. The following example lists the Classic Load Balancers for *my\-asg*\. 

```
aws autoscaling describe-load-balancers --auto-scaling-group-name my-asg
```

### Detach your target group or Classic Load Balancer \(legacy\)<a name="example-detach-load-balancer-target-group"></a>

**To detach a target group**  
The following [detach\-load\-balancer\-target\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancer-target-groups.html) command detaches a target group from your Auto Scaling group when you no longer need it\. 

```
aws autoscaling detach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns "arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/1234567890123456"
```

**To detach a Classic Load Balancer**  
The following [detach\-load\-balancers](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancers.html) command detaches a Classic Load Balancer from your Auto Scaling group when you no longer need it\.

```
aws autoscaling detach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```