# Deleting a Scaling Policy<a name="deleting-scaling-policy"></a>

After you no longer need a scaling policy, you can delete it\. You might also need to delete the CloudWatch alarms\. Deleting a target tracking scaling policy also deletes any associated CloudWatch alarms\. Deleting a step scaling policy or a simple scaling policy deletes the underlying alarm action, but does not delete the CloudWatch alarm associated with the scaling policy, even if it no longer has an associated action\. 

**To delete a scaling policy \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select the Auto Scaling group\.

1. On the **Scaling Policies** tab, choose **Actions**, **Delete**\. 

1. When prompted for confirmation, choose **Yes, Delete**\.

1. \(Optional\) If you deleted a step scaling policy or a simple scaling policy, do the following to delete the CloudWatch alarm that was associated with the policy\. You can skip this step to keep the alarm for future use\.

   1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

   1. On the navigation pane, choose **Alarms**\.

   1. Choose the alarm \(for example, `awsec2-my-auto-scaling-group-CPU-Utilization`\) and choose **Action**, **Delete**\.

   1. When prompted for confirmation, choose **Delete**\.

**To delete your scaling policy \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-policy.html) command\. 

```
aws autoscaling delete-policy --auto-scaling-group-name my-asg \
  --policy-name my-step-scalein-policy
```

**To delete your CloudWatch alarm \(AWS CLI\)**  
For step scaling policies and simple scaling policies, use the [https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/delete-alarms.html](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/delete-alarms.html) command to delete the CloudWatch alarm that was associated with the policy\. You can skip this step to keep the alarm for future use\. You can delete one or more alarms at a time\. For example, use the following command to delete the `Step-Scaling-AlarmHigh-AddCapacity` and `Step-Scaling-AlarmLow-RemoveCapacity` alarms\.

```
aws cloudwatch delete-alarms --alarm-name Step-Scaling-AlarmHigh-AddCapacity Step-Scaling-AlarmLow-RemoveCapacity
```