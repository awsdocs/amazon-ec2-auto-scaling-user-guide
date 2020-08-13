# Deleting your Auto Scaling infrastructure<a name="as-process-shutdown"></a>

To completely delete your scaling infrastructure, complete the following tasks\.

**Topics**
+ [Delete your Auto Scaling group](#as-shutdown-lbs-delete-asg-cli)
+ [\(Optional\) Delete the launch configuration](#as-shutdown-lbs-delete-lc-cli)
+ [\(Optional\) Delete the launch template](#as-shutdown-lbs-delete-lt-cli)
+ [\(Optional\) Delete the load balancer and target groups](#as-shutdown-lbs-delete-lbs-cli)
+ [\(Optional\) Delete CloudWatch alarms](#as-shutdown-delete-alarms-cli)

## Delete your Auto Scaling group<a name="as-shutdown-lbs-delete-asg-cli"></a>

When you delete an Auto Scaling group, its desired, minimum, and maximum values are set to 0\. As a result, the instances are terminated\. Deleting an instance also deletes any associated logs or data, and any volumes on the instance\. If do not want to terminate one or more instances, you can detach them before you delete the Auto Scaling group\. If the group has scaling policies, deleting the group deletes the policies, the underlying alarm actions, and any alarm that no longer has an associated action\.

**To delete your Auto Scaling group \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. On the **Auto Scaling Groups** page, select the check box next to your Auto Scaling group and choose **Delete** \(Old console: choose **Actions**, **Delete**\)\. 

1. When prompted for confirmation, choose **Delete**\.

   A loading icon in the **Name** column indicates that the Auto Scaling group is being deleted\. The **Desired**, **Min**, and **Max** columns show `0` instances for the Auto Scaling group\. It takes a few minutes to terminate the instance and delete the group\. Refresh the list to see the current state\. 

**To delete your Auto Scaling group \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html) command to delete the Auto Scaling group\. 

```
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name my-asg
```

If the group has instances or scaling activities in progress, use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html) command with the `--force-delete` option\. This will also terminate the Amazon EC2 instances\.

```
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name my-asg --force-delete
```

## \(Optional\) Delete the launch configuration<a name="as-shutdown-lbs-delete-lc-cli"></a>

You can skip this step to keep the launch configuration for future use\.

**To delete the launch configuration \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Launch Configurations**\.

1. On the **Launch Configurations** page, choose your launch configuration and choose **Actions**, **Delete launch configuration**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete the launch configuration \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-launch-configuration.html) command\.

```
aws autoscaling delete-launch-configuration --launch-configuration-name my-launch-config
```

## \(Optional\) Delete the launch template<a name="as-shutdown-lbs-delete-lt-cli"></a>

You can delete your launch template or just one version of your launch template\. When you delete a launch template, all its versions are deleted\.

You can skip this step to keep the launch template for future use\. 

**To delete your launch template \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **INSTANCES**, choose **Launch Templates**\.

1. Select your launch template and then do one of the following: 
   + Choose **Actions**, **Delete template**\. When prompted for confirmation, choose **Delete launch template**\.
   + Choose **Actions**, **Delete template version**\. Select the version to delete and choose **Delete launch template version**\.

**To delete the launch template \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template.html) command to delete your template and all its versions\.

```
aws ec2 delete-launch-template --launch-template-id lt-068f72b72934aff71
```

Alternatively, you can use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template-versions.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template-versions.html) command to delete a specific version of a launch template\. 

```
aws ec2 delete-launch-template-versions --launch-template-id lt-068f72b72934aff71 --versions 1
```

## \(Optional\) Delete the load balancer and target groups<a name="as-shutdown-lbs-delete-lbs-cli"></a>

Skip this step if your Auto Scaling group is not associated with an Elastic Load Balancing load balancer, or if you want to keep the load balancer for future use\. 

**To delete your load balancer \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Choose the load balancer and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete your target group \(console\)**

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the target group and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes**\.

**To delete the load balancer associated with the Auto Scaling group \(AWS CLI\)**  
For Application Load Balancers and Network Load Balancers, use the following [https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-load-balancer.html](https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-load-balancer.html) and [https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-target-group.html](https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-target-group.html) commands\.

```
aws elbv2 delete-load-balancer --load-balancer-arn my-load-balancer-arn
aws elbv2 delete-target-group --target-group-arn my-target-group-arn
```

For Classic Load Balancers, use the following [https://docs.aws.amazon.com/cli/latest/reference/elb/delete-load-balancer.html](https://docs.aws.amazon.com/cli/latest/reference/elb/delete-load-balancer.html) command\.

```
aws elb delete-load-balancer --load-balancer-name my-load-balancer
```

## \(Optional\) Delete CloudWatch alarms<a name="as-shutdown-delete-alarms-cli"></a>

To delete any CloudWatch alarms associated with your Auto Scaling group, complete the following steps\. 

You can skip this step if your Auto Scaling group is not associated with any CloudWatch alarms, or if you want to keep the alarms for future use\.

**Note**  
Deleting an Auto Scaling group automatically deletes the CloudWatch alarms that Amazon EC2 Auto Scaling manages for a target tracking scaling policy\. 

**To delete the CloudWatch alarms \(console\)**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation pane, choose **Alarms**\.

1. Choose the alarms and choose **Action**, **Delete**\.

1. When prompted for confirmation, choose **Delete**\.

**To delete the CloudWatch alarms \(AWS CLI\)**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/delete-alarms.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/delete-alarms.html) command\. You can delete one or more alarms at a time\. For example, use the following command to delete the `Step-Scaling-AlarmHigh-AddCapacity` and `Step-Scaling-AlarmLow-RemoveCapacity` alarms\.

```
aws cloudwatch delete-alarms --alarm-name Step-Scaling-AlarmHigh-AddCapacity Step-Scaling-AlarmLow-RemoveCapacity
```