# Set capacity limits on your Auto Scaling group<a name="asg-capacity-limits"></a>

Capacity limits represent the minimum and maximum group size that you want for your Auto Scaling group\. You set limits separately for the minimum and maximum size\.

The group's desired capacity can be resized to a number that's within the range of your minimum and maximum size limits\. The desired capacity must be equal to or greater than the minimum group size, and equal to or less than the maximum group size\.
+ **Desired capacity**: Represents the initial capacity of the Auto Scaling group at the time of creation\. An Auto Scaling group attempts to maintain the desired capacity\. It starts by launching the number of instances that are specified for the desired capacity, and maintains this number of instances as long as there are no scaling policies or scheduled actions attached to the Auto Scaling group\.
+ **Minimum capacity**: Represents the minimum group size\. When scaling policies are set, an Auto Scaling group cannot decrease its desired capacity lower than the minimum size limit\.
+ **Maximum capacity**: Represents the maximum group size\. When scaling policies are set, an Auto Scaling group cannot increase its desired capacity higher than the maximum size limit\.

The minimum and maximum size limits also apply in the following scenarios:
+ When you manually scale your Auto Scaling group by updating its desired capacity\.
+ When scheduled actions run that update the desired capacity\. If a scheduled action runs without specifying new minimum and maximum size limits for the group, then the group's current minimum and maximum size limits apply\.

An Auto Scaling group always tries to maintain its desired capacity\. In cases where an instance terminates unexpectedly \(for example, because of a Spot Instance interruption, a health check failure, or human action\), the group automatically launches a new instance to maintain its desired capacity\.

**To manage these settings in the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\. 

1. On the **Auto Scaling groups** page, select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom of the page\. 

1. In the lower pane, in the **Details** tab, view or change the current settings for minimum, maximum, and desired capacity\. For more information, see [Change the size of your Auto Scaling group \(console\)](as-manual-scaling.md#as-manual-scaling-console)\.

Above the **Details** pane, you can find information such as the current number of instances in the Auto Scaling group, the minimum, maximum, and desired capacity, and a status column\. If the Auto Scaling group uses instance weighting, you can also find the number of capacity units contributed to the desired capacity\.

To add or remove columns from the list, choose the settings icon at the top of the page\. Then, for **Auto Scaling groups attributes**, turn each column on or off, and choose **Confirm**\. 

**To verify the size of your Auto Scaling group after making changes**  
The **Instances** column shows the number of instances that are currently running\. While an instance is being launched or terminated, the **Status** column displays a status of *Updating capacity*, as shown in the following image\. 

![\[Updating the capacity of an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/asg-console-updating-capacity.png)![\[Updating the capacity of an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[Updating the capacity of an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

Wait for a few minutes, and then refresh the view to see the latest status\. After a scaling activity completes, the **Instances** column shows an updated value\. 

You can also view the number of instances and the status of the currently running instances from the **Instance management** tab, under **Instances**\.

**Note**  
You can also use Service Quotas to update the total capacity limits for EC2 instances and other resources in your AWS account\. In the Service Quotas console, you can view all your available service quotas and request increases for them\. For more information, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html) in the *Service Quotas User Guide*\.