# Common termination policy scenarios for Amazon EC2 Auto Scaling<a name="common-scenarios-termination"></a>

The following are common termination policy scenarios for Amazon EC2 Auto Scaling instance termination\. By default, Amazon EC2 Auto Scaling uses the default termination policy, but you can optionally specify a termination policy of your own\. 

**Topics**
+ [Scale\-in events](#common-scenarios-termination-scale-in)
+ [Rebalancing activities](#common-scenarios-termination-rebalancing)
+ [Instance refreshes](#common-scenarios-termination-instance-types)

## Scale\-in events<a name="common-scenarios-termination-scale-in"></a>

The most common scenario for using a termination policy is when Amazon EC2 Auto Scaling launches Amazon EC2 instances using scaling policies, and then terminates those instances when they are no longer needed as part of a scale\-in event\. A scale\-in event can also occur because of a scheduled action or where there is a new value for desired capacity that is lower than the current capacity of the group\.

Consider an Auto Scaling group that has one instance type, two Availability Zones, a desired capacity of two instances, and a scaling policy that adds and removes instances as resource utilization increases and decreases\. The two instances in this group are distributed as follows\.

![\[A basic Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/termination-policy-default-diagram.png)

When the Auto Scaling group scales out, Amazon EC2 Auto Scaling launches a new instance\. The Auto Scaling group now has three instances, distributed as follows\.

![\[An Auto Scaling group after a scaling action occurs.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/termination-policy-default-2-diagram.png)

When the Auto Scaling group scales in, Amazon EC2 Auto Scaling terminates one of the instances\. 

If you did not assign a specific termination policy to the group, it uses the default termination policy\. It selects the Availability Zone with two instances, and terminates the instance that was launched from the oldest launch configuration\. If the instances were launched from the same launch configuration, Amazon EC2 Auto Scaling selects the instance that is closest to the next billing hour and terminates it\. 

Note that the default termination policy works for Auto Scaling groups created with a launch template or with a launch configuration\. You can move your Auto Scaling groups from using launch configurations to launch templates at any time and continue to use the default termination policy\. The default termination policy will continue to terminate instances launched from the oldest launch configuration until there are no more remaining instances created from a launch configuration\. After that, it terminates instances launched from the oldest launch template\.

## Rebalancing activities<a name="common-scenarios-termination-rebalancing"></a>

[Rebalancing activities](auto-scaling-benefits.md#AutoScalingBehavior.InstanceUsage) occur to proactively balance your instances across Availability Zones evenly for high availability\. For example, rebalancing can be necessary when there's an availability outage, when there are changes to the Availability Zones, and when you remove instances\. When terminating instances due to rebalancing activities, the termination policy determines which instances are terminated\. 

### Availability outage<a name="common-scenarios-termination-outage"></a>

Availability outages are rare\. However, if one Availability Zone becomes unavailable, and then recovers, your Auto Scaling group can become unbalanced between Availability Zones\. Amazon EC2 Auto Scaling then tries to gradually rebalance the group, and rebalancing might terminate instances in other zones\.

Take the example where you have an Auto Scaling group that has one instance type, two Availability Zones, and a desired capacity of two instances\. In a situation where one Availability Zone fails, Amazon EC2 Auto Scaling automatically launches a new instance in the healthy Availability Zone to replace the one in the unhealthy Availability Zone\.

When the unhealthy Availability Zone returns to a healthy state, Amazon EC2 Auto Scaling automatically launches a new instance in this zone, which in turn terminates an instance in the unaffected zone\. 

### Changes to Availability Zones<a name="common-scenarios-termination-new-zone"></a>

An existing Auto Scaling group can be updated to add more subnets, either for existing Availability Zones or for new Availability Zones that have been added since the creation of the Auto Scaling group\. If you expand your Auto Scaling group to include additional Availability Zones, or you change which Availability Zones are used, Amazon EC2 Auto Scaling will launch instances in the new Availability Zones and terminate instances in the other zones to help ensure that your instances span Availability Zones evenly\.

### Removing instances<a name="common-scenarios-termination-removed-instances"></a>

If you detach instances from your Auto Scaling group, or you explicitly terminate instances and decrement the desired capacity, thereby preventing replacement instances from launching, the group can become unbalanced\. If this occurs, Amazon EC2 Auto Scaling compensates by rebalancing the Availability Zones\.

## Instance refreshes<a name="common-scenarios-termination-instance-types"></a>

During an instance refresh, which you start in order to update the instances in your Auto Scaling group, Amazon EC2 Auto Scaling terminates instances in the group and then launches replacements for the terminated instances\. 

For example, let's say that you changed the instance types that are associated with your Auto Scaling group\. After making the change, you start an instance refresh to force replacement instances to launch that use your new instance types\. If you run an instance refresh on a group of 10 instances and set the minimum healthy percentage to 90%, Amazon EC2 Auto Scaling replaces one instance before continuing on to replace the next instance\. The termination policy controls which instance is replaced first\. 