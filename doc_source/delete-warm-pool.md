# Deleting a warm pool<a name="delete-warm-pool"></a>

When you no longer need the warm pool, use the following procedure to delete it\.

**To delete your warm pool \(console\)**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Delete**\. 

**To delete your warm pool \(AWS CLI\)**  
Use the following [delete\-warm\-pool](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-warm-pool.html) command to delete the warm pool\. 

```
aws autoscaling delete-warm-pool --auto-scaling-group-name my-asg
```

If there are instances in the warm pool, or if scaling activities are in progress, use the [delete\-warm\-pool](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-warm-pool.html) command with the `--force-delete` option\. This option will also terminate the Amazon EC2 instances and any outstanding lifecycle actions\.

```
aws autoscaling delete-warm-pool --auto-scaling-group-name my-asg --force-delete
```