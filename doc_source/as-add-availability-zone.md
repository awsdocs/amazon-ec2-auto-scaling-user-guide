# Expanding your scaled and load\-balanced application to an additional Availability Zone<a name="as-add-availability-zone"></a>

You can take advantage of the safety and reliability of geographic redundancy by spanning your Auto Scaling group across multiple Availability Zones within a Region and then attaching a load balancer to distribute incoming traffic across those Zones\. Incoming traffic is distributed equally across all Availability Zones enabled for your load balancer\.

When one Availability Zone becomes unhealthy or unavailable, Amazon EC2 Auto Scaling launches new instances in an unaffected Zone\. When the unhealthy Availability Zone returns to a healthy state, Amazon EC2 Auto Scaling automatically redistributes the application instances evenly across all of the Zones for your Auto Scaling group\. Amazon EC2 Auto Scaling does this by attempting to launch new instances in the Availability Zone with the fewest instances\. If the attempt fails, however, Amazon EC2 Auto Scaling attempts to launch in other Availability Zones until it succeeds\.

You can expand the availability of your scaled and load\-balanced application by adding an Availability Zone to your Auto Scaling group and then enabling that Zone for your load balancer\. After you've enabled the new Availability Zone, the load balancer begins to route traffic equally among all the enabled Zones\. 

**Limits**
+ An Auto Scaling group can contain Amazon EC2 instances from multiple Availability Zones within the same Region\. However, an Auto Scaling group can't contain instances from multiple Regions\.
+ To change which Availability Zones are enabled for your load balancer, you need to be aware of the following limitations: 
  + When you enable an Availability Zone for your load balancer, you specify one subnet from that Availability Zone\. Note that you can select at most one subnet per Availability Zone\. 
  + For internet\-facing load balancers, the subnets that you specify for the load balancer must have at least 8 available IP addresses\. 
  + For Application Load Balancers, you must enable at least two Availability Zones\.
  + For Network Load Balancers, you cannot disable the enabled Availability Zones, but you can enable additional ones\.
  + For Gateway Load Balancers, you cannot change the Availability Zones or subnets that were added when the load balancer was created\.

**Topics**
+ [Add an Availability Zone \(console\)](#as-add-az-console)
+ [Add an Availability Zone \(AWS CLI\)](#as-add-az-aws-cli)

## Add an Availability Zone \(console\)<a name="as-add-az-console"></a>

Use the following procedure to expand your Auto Scaling group and load balancer to a subnet in an additional Availability Zone\.

**To add an Availability Zone**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Network**, **Edit**\.

1. In **Subnets**, choose the subnet corresponding to the Availability Zone\.

1. Choose **Update**\.

1. To update the Availability Zones for your load balancer so that it shares the same Zones as your Auto Scaling group, complete the following steps:

   1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

   1. Choose your load balancer\.

   1. Do one of the following:
      + For Application Load Balancers and Network Load Balancers:

        1. On the **Description** tab, for **Availability Zones**, choose **Edit subnets**\. 

        1. On the **Edit subnets** page, for **Availability Zones**, select the check box for the Availability Zone to add, and then select one of the subnets for that Availability Zone\.
      + For Classic Load Balancers in a VPC:

        1. On the **Instances** tab, choose **Edit Availability Zones**\. 

        1. On the **Add and Remove Subnets** page, for **Available subnets**, choose the add icon \(\+\) for the subnet to add\. The subnet is moved under **Selected subnets**\.
      + For Classic Load Balancers in EC2\-Classic:

        1. On the **Instances** tab, choose **Edit Availability Zones**\. 

        1. On the **Add and Remove Availability Zones** page, choose the Availability Zone to add\.

   1. Choose **Save**\.

## Add an Availability Zone \(AWS CLI\)<a name="as-add-az-aws-cli"></a>

The commands that you use depend on whether your load balancer is an Application Load Balancer or Network Load Balancer, a Classic Load Balancer in a VPC, a Classic Load Balancer in EC2\-Classic\.

**For an Auto Scaling group with an Application Load Balancer or Network Load Balancer**

1. Add a subnet to the Auto Scaling group using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --vpc-zone-identifier subnet-41767929 subnet-cb663da2 subnet-8360a9e7
   ```

1. Verify that the instances in the new subnet are ready to accept traffic from the load balancer using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Enable the new subnet for your Application Load Balancer or Network Load Balancer using the following [https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-subnets.html](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-subnets.html) command\.

   ```
   aws elbv2 set-subnets --load-balancer-arn my-lb-arn \
     --subnets subnet-41767929 subnet-cb663da2 subnet-8360a9e7
   ```

**For an Auto Scaling group with a Classic Load Balancer in a VPC**

1. Add a subnet to the Auto Scaling group using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --vpc-zone-identifier subnet-41767929 subnet-cb663da2
   ```

1. Verify that the instances in the new subnet are ready to accept traffic from the load balancer using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Enable the new subnet for your Classic Load Balancer using the following [https://docs.aws.amazon.com/cli/latest/reference/elb/attach-load-balancer-to-subnets.html](https://docs.aws.amazon.com/cli/latest/reference/elb/attach-load-balancer-to-subnets.html) command\.

   ```
   aws elb attach-load-balancer-to-subnets --load-balancer-name my-lb \
     --subnets subnet-41767929
   ```

**For an Auto Scaling group with a Classic Load Balancer in EC2\-Classic**

1. Add an Availability Zone to the Auto Scaling group using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
     --availability-zones us-west-2a us-west-2b us-west-2c
   ```

1. Verify that the instances in the new Availability Zone are ready to accept traffic from the load balancer using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Enable the new Availability Zone for your Classic Load Balancer using the following [https://docs.aws.amazon.com/cli/latest/reference/elb/enable-availability-zones-for-load-balancer.html](https://docs.aws.amazon.com/cli/latest/reference/elb/enable-availability-zones-for-load-balancer.html) command\.

   ```
   aws elb enable-availability-zones-for-load-balancer --load-balancer-name my-lb \
     --availability-zones us-west-2c
   ```