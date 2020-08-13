# Setting capacity limits for your Auto Scaling group<a name="asg-capacity-limits"></a>

You configure the size of your Auto Scaling group by setting the minimum, maximum, and desired capacity\. The minimum and maximum capacity are required to create an Auto Scaling group, while the desired capacity is optional\. If you do not define your desired capacity up front, it defaults to your minimum capacity\. 

**Note**  
By default, the minimum, maximum, and desired capacity are set to one instance when you create an Auto Scaling group from the console\. If you change the desired capacity, the capacity that you specify will be the total number of instances launched right after creating your Auto Scaling group\.

An Auto Scaling group is elastic as long as it has different values for minimum and maximum capacity\. All requests to change the Auto Scaling group's desired capacity \(either by manual scaling or automatic scaling\) must fall within these limits\.

If you choose to automatically scale your group, the **maximum** limit lets Amazon EC2 Auto Scaling scale out the number of instances as needed to handle an increase in demand\. The **minimum** limit helps ensure that you always have a certain number of instances running at all times\. 

These limits also apply when you manually scale your Auto Scaling group, such as when you want to turn off automatic scaling and have the group run at a fixed size, either temporarily or permanently\.

**To access capacity settings in the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\. 

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, view or change the current settings for minimum, maximum, and desired capacity\.