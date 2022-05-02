# Set capacity limits on your Auto Scaling group<a name="asg-capacity-limits"></a>

Capacity limits place restrictions on the size of your Auto Scaling group\. You set limits separately for the minimum and maximum size\. The group's desired capacity is resizeable between the minimum and maximum size limits\. The desired capacity must be greater than or equal to the minimum size of the group and less than or equal to the maximum size of the group\.

An Auto Scaling group will start by launching as many instances as are specified for desired capacity\. If there are no scaling policies or scheduled actions attached to the Auto Scaling group, Amazon EC2 Auto Scaling maintains the desired amount of instances, performing periodic health checks on the instances in the group\. Unhealthy instances will be terminated and replaced with new ones\.

If you choose to turn on auto scaling, the maximum limit lets Amazon EC2 Auto Scaling scale out the number of instances as needed to handle an increase in demand\. The minimum limit helps ensure that you always have a certain number of instances running at all times\. 

The minimum and maximum size limits also apply when you manually scale your Auto Scaling group, such as when you want to turn off auto scaling and have the group run at a fixed size, either temporarily or permanently\. In this case, you can manage the size of the Auto Scaling group by updating its desired capacity as needed\.

**To manage these settings in the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\. 

1. On the **Auto Scaling groups** page, select the check box next to the Auto Scaling group whose settings you want to manage\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. In the lower pane, in the **Details** tab, view or change the current settings for minimum, maximum, and desired capacity\.

Above the **Details** pane contains overview information about the Auto Scaling group, including the current number of instances in the group, the minimum, maximum, and desired capacity, and a status column\. If the Auto Scaling group uses instance weighting, then the information includes the number of capacity units contributed to the desired capacity\. To add or remove columns from the list, choose the settings icon at the top of the page\. Then, for **Auto Scaling groups attributes**, turn each column on or off, and choose **Confirm** changes\. 

**To verify the size of your Auto Scaling group after making changes**  
The **Instances** column shows the number of instances that are currently running\. While an instance is being launched or terminated, the **Status** column displays a status of *Updating capacity*, as shown in the following image\. 

![\[Updating the capacity of an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/asg-console-updating-capacity.png)![\[Updating the capacity of an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[Updating the capacity of an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

Wait for a few minutes, and then refresh the view to see the latest status\. After a scaling activity completes, notice that the **Instances** column shows a new value\. 

You can also view the number of instances and the status of the instances that are currently running from the **Instance management** tab under **Instances**\.