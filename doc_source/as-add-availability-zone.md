# Add and remove Availability Zones<a name="as-add-availability-zone"></a>

To take advantage of the safety and reliability of geographic redundancy, span your Auto Scaling group across multiple Availability Zones of the Region you are working in and attach a load balancer to distribute incoming traffic across those Availability Zones\. 

When one Availability Zone becomes unhealthy or unavailable, Amazon EC2 Auto Scaling launches new instances in an unaffected Availability Zone\. When the unhealthy Availability Zone returns to a healthy state, Amazon EC2 Auto Scaling automatically redistributes the application instances evenly across all the Availability Zones for your Auto Scaling group\. Amazon EC2 Auto Scaling does this by attempting to launch new instances in the Availability Zone with the fewest instances\. If the attempt fails, however, Amazon EC2 Auto Scaling attempts to launch in other Availability Zones until it succeeds\.

Elastic Load Balancing creates a load balancer node for each Availability Zone you enable for the load balancer\. If you enable cross\-zone load balancing for your load balancer, each load balancer node distributes traffic evenly across the registered instances in all enabled Availability Zones\. If cross\-zone load balancing is disabled, each load balancer node distributes requests evenly across the registered instances in its Availability Zone only\. 

You must specify at least one Availability Zone when you are creating your Auto Scaling group\. Later, you can expand the availability of your application by adding an Availability Zone to your Auto Scaling group and enabling that Availability Zone for your load balancer \(if the load balancer supports it\)\.

**Topics**
+ [Add an Availability Zone](#as-add-az-console)
+ [Remove an Availability Zone](#as-remove-az-console)
+ [Limitations](#availability-zone-limitations)

## Add an Availability Zone<a name="as-add-az-console"></a>

Use the following procedure to expand your Auto Scaling group and load balancer to a subnet in an additional Availability Zone\.

**To add an Availability Zone**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Network**, **Edit**\.

1. In **Subnets**, choose the subnet corresponding to the Availability Zone that you want to add to the Auto Scaling group\.

1. Choose **Update**\.

1. To update the Availability Zones for your load balancer so that it shares the same Availability Zones as your Auto Scaling group, complete the following steps:

   1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

   1. Choose your load balancer\.

   1. Do one of the following:
      + For Application Load Balancers and Network Load Balancers:

        1. On the **Description** tab, for **Availability Zones**, choose **Edit subnets**\. 

        1. On the **Edit subnets** page, for **Availability Zones**, select the check box for the Availability Zone to add\. If there is only one subnet for that zone, it is selected\. If there is more than one subnet for that zone, select one of the subnets\. 
      + For Classic Load Balancers in a VPC:

        1. On the **Instances** tab, choose **Edit Availability Zones**\. 

        1. On the **Add and Remove Subnets** page, for **Available subnets**, select the subnet using its add \(\+\) icon\. The subnet is moved under **Selected subnets**\.

   1. Choose **Save**\.

## Remove an Availability Zone<a name="as-remove-az-console"></a>

To remove an Availability Zone from your Auto Scaling group and load balancer, use the following procedure\.

**To remove an Availability Zone**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Network**, **Edit**\.

1. In **Subnets**, choose the delete \(X\) icon for the subnet corresponding to the Availability Zone that you want to remove from the Auto Scaling group\. If there is more than one subnet for that zone, choose the delete \(X\) icon for each one\. 

1. Choose **Update**\.

1. To update the Availability Zones for your load balancer so that it shares the same Availability Zones as your Auto Scaling group, complete the following steps:

   1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

   1. Choose your load balancer\.

   1. Do one of the following:
      + For Application Load Balancers and Network Load Balancers:

        1. On the **Description** tab, for **Availability Zones**, choose **Edit subnets**\. 

        1. On the **Edit subnets** page, for **Availability Zones**, clear the check box to remove the subnet for that Availability Zone\.
      + For Classic Load Balancers in a VPC:

        1. On the **Instances** tab, choose **Edit Availability Zones**\. 

        1. On the **Add and Remove Subnets** page, for **Available subnets**, remove the subnet using its delete \(\-\) icon\. The subnet is moved under **Available subnets**\.

   1. Choose **Save**\.

## Limitations<a name="availability-zone-limitations"></a>

To update which Availability Zones are enabled for your load balancer, you need to be aware of the following limitations: 
+ When you enable an Availability Zone for your load balancer, you specify one subnet from that Availability Zone\. Note that you can enable at most one subnet per Availability Zone for your load balancer\. 
+ For internet\-facing load balancers, the subnets that you specify for the load balancer must have at least eight available IP addresses\. 
+ For Application Load Balancers, you must enable at least two Availability Zones\.
+ For Network Load Balancers, you cannot disable the enabled Availability Zones, but you can enable additional ones\.
+ For Gateway Load Balancers, you cannot change the Availability Zones or subnets that were added when the load balancer was created\.