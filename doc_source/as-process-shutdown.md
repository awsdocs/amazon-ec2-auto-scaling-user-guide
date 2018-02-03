# Deleting Your Auto Scaling Infrastructure<a name="as-process-shutdown"></a>

To completely delete your scaling infrastructure, complete the following tasks\.


+ [Delete Your Auto Scaling Group](#as-shutdown-lbs-delete-asg-cli)
+ [\(Optional\) Delete the Launch Configuration](#as-shutdown-lbs-delete-lc-cli)
+ [\(Optional\) Delete the Load Balancer](#as-shutdown-lbs-delete-lbs-cli)
+ [\(Optional\) Delete CloudWatch Alarms](#as-shutdown-delete-alarms-cli)

## Delete Your Auto Scaling Group<a name="as-shutdown-lbs-delete-asg-cli"></a>

When you delete an Auto Scaling group, its desired, minimum, and maximum values are set to 0\. As a result, the Auto Scaling instances are terminated\. Alternatively, you can terminate or detach the instances before you delete the Auto Scaling group\.

**To delete your Auto Scaling group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. On the Auto Scaling groups page, select your Auto Scaling group\. and choose **Actions**, **Delete**\. 

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete your Auto Scaling group using the AWS CLI**  
Use the following [delete\-auto\-scaling\-group ](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html) command to delete the Auto Scaling group:

```
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name my-asg
```

## \(Optional\) Delete the Launch Configuration<a name="as-shutdown-lbs-delete-lc-cli"></a>

Note that you can skip this step if you want to keep the launch configuration for future use\.

**To delete the launch configuration using the console**

1. On the navigation pane, under **Auto Scaling**, choose **Launch Configurations**\.

1. On the **Launch Configurations** page, select your launch configuration and choose **Actions**, **Delete launch configuration**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete the launch configuration using the AWS CLI**  
Use the following [delete\-launch\-configuration](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-launch-configuration.html) command:

```
aws autoscaling delete-launch-configuration --launch-configuration-name my-lc
```

## \(Optional\) Delete the Load Balancer<a name="as-shutdown-lbs-delete-lbs-cli"></a>

Note that you can skip this step if your Auto Scaling group is not registered with an Elastic Load Balancing load balancer or you want to keep the load balancer for future use\.

**To delete your load balancer**

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete the load balancer associated with the Auto Scaling group using the AWS CLI**  
For Application Load Balancers and Network Load Balancers, use the following [delete\-load\-balancer](http://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-load-balancer.html) command:

```
aws elbv2 delete-load-balancer --load-balancer-arn my-load-balancer-arn
```

For Classic Load Balancers, use the following [delete\-load\-balancer](http://docs.aws.amazon.com/cli/latest/reference/elb/delete-load-balancer.html) command:

```
aws elb delete-load-balancer --load-balancer-name my-load-balancer
```

## \(Optional\) Delete CloudWatch Alarms<a name="as-shutdown-delete-alarms-cli"></a>

Note that you can skip this step if your Auto Scaling group is not associated with any CloudWatch alarms or you want to keep the alarms for future use\.

**To delete the CloudWatch alarms using the console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation pane, choose **Alarms**\.

1. Select the alarms and choose **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete the CloudWatch alarms using the AWS CLI**  
Use the [delete\-alarms](http://docs.aws.amazon.com/cli/latest/reference/cloudwatch/delete-alarms.html) command\. For example, use the following command to delete the `AddCapacity` and `RemoveCapacity` alarms:

```
aws cloudwatch delete-alarms --alarm-name AddCapacity RemoveCapacity
```