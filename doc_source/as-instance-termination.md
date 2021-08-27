# Controlling which Auto Scaling instances terminate during scale in<a name="as-instance-termination"></a>

Amazon EC2 Auto Scaling uses termination policies to determine which instances it terminates first during scale\-in events\. Termination policies define the termination criteria that is used by Amazon EC2 Auto Scaling when choosing which instances to terminate\.

Amazon EC2 Auto Scaling uses a default termination policy, but you can optionally choose or create your own termination policies with your own termination criteria\. This lets you ensure that your instances are terminated based on your specific application needs\.

Amazon EC2 Auto Scaling also provides instance scale\-in protection\. When you enable this feature, it prevents instances from being terminated during scale\-in events\. You can enable instance scale\-in protection when you create an Auto Scaling group, and you can change the setting on running instances\. If you enable instance scale\-in protection on an existing Auto Scaling group, all new instances launched after that will have instance scale\-in protection enabled\.

**Note**  
Instance scale\-in protection does not guarantee that instances won't be terminated in the event of a human error—for example, if someone manually terminates an instance using the Amazon EC2 console or AWS CLI\. To protect your instance from accidental termination, you can use Amazon EC2 termination protection\. However, even with termination protection and instance scale\-in protection enabled, data saved to instance storage can be lost if a health check determines that an instance is unhealthy or if the group itself is accidentally deleted\. As with any environment, a best practice is to back up your data frequently, or whenever it's appropriate for your business continuity requirements\. 

**Topics**
+ [Scenarios for termination policy use](#common-scenarios-termination)
+ [Working with Amazon EC2 Auto Scaling termination policies](ec2-auto-scaling-termination-policies.md)
+ [Creating a custom termination policy with Lambda](lambda-custom-termination-policy.md)
+ [Using instance scale\-in protection](ec2-auto-scaling-instance-protection.md)

## Scenarios for termination policy use<a name="common-scenarios-termination"></a>

The following sections describe the scenarios in which Amazon EC2 Auto Scaling uses termination policies\. 

**Topics**
+ [Scale\-in events](#common-scenarios-termination-scale-in)
+ [Instance refreshes](#common-scenarios-termination-instance-refreshes)
+ [Availability Zone rebalancing](#common-scenarios-termination-rebalancing)

### Scale\-in events<a name="common-scenarios-termination-scale-in"></a>

A scale\-in event occurs when there is a new value for the desired capacity of an Auto Scaling group that is lower than the current capacity of the group\.

Scale\-in events occur in the following scenarios:
+ When using dynamic scaling policies and the size of the group decreases as a result of changes in a metric's value
+ When using scheduled scaling and the size of the group decreases as a result of a scheduled action
+ When you manually decrease the size of the group

The following example shows how termination policies work when there is a scale\-in event\.

1. The Auto Scaling group in this example has one instance type, two Availability Zones, and a desired capacity of two instances\. It also has a dynamic scaling policy that adds and removes instances when resource utilization increases or decreases\. The two instances in this group are distributed across the two Availability Zones as shown in the following diagram\.  
![\[A basic Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/termination-policy-default-diagram.png)

1. When the Auto Scaling group scales out, Amazon EC2 Auto Scaling launches a new instance\. The Auto Scaling group now has three instances, distributed across the two Availability Zones as shown in the following diagram\.  
![\[An Auto Scaling group after a scaling action occurs.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/termination-policy-default-2-diagram.png)

1. When the Auto Scaling group scales in, Amazon EC2 Auto Scaling terminates one of the instances\. 

1. If you did not assign a specific termination policy to the group, Amazon EC2 Auto Scaling uses the default termination policy\. It selects the Availability Zone with two instances, and terminates the instance that was launched from the oldest launch template or launch configuration\. If the instances were launched from the same launch template or launch configuration, Amazon EC2 Auto Scaling selects the instance that is closest to the next billing hour and terminates it\. 

### Instance refreshes<a name="common-scenarios-termination-instance-refreshes"></a>

You start instance refreshes in order to update the instances in your Auto Scaling group\. During an instance refresh, Amazon EC2 Auto Scaling terminates instances in the group and then launches replacements for the terminated instances\. The termination policy for the Auto Scaling group controls which instances are replaced first\. 

### Availability Zone rebalancing<a name="common-scenarios-termination-rebalancing"></a>

Amazon EC2 Auto Scaling balances your capacity evenly across the Availability Zones enabled for your Auto Scaling group\. This helps reduce the impact of an Availability Zone outage\. If the distribution of capacity across Availability Zones becomes out of balance, Amazon EC2 Auto Scaling rebalances the Auto Scaling group by launching instances in the enabled Availability Zones with the fewest instances and terminating instances elsewhere\. The termination policy controls which instances are prioritized for termination first\. 

There are a number of reasons why the distribution of instances across Availability Zones can become out of balance\.

Removing instances  
If you detach instances from your Auto Scaling group, you put instances on standby, or you explicitly terminate instances and decrement the desired capacity, which prevents replacement instances from launching, the group can become unbalanced\. If this occurs, Amazon EC2 Auto Scaling compensates by rebalancing the Availability Zones\.

Using different Availability Zones than originally specified  
If you expand your Auto Scaling group to include additional Availability Zones, or you change which Availability Zones are used, Amazon EC2 Auto Scaling launches instances in the new Availability Zones and terminates instances in other zones to help ensure that your Auto Scaling group spans Availability Zones evenly\.

Availability outage  
Availability outages are rare\. However, if one Availability Zone becomes unavailable and recovers later, your Auto Scaling group can become unbalanced between Availability Zones\. Amazon EC2 Auto Scaling tries to gradually rebalance the group, and rebalancing might terminate instances in other zones\.  
For example, imagine that you have an Auto Scaling group that has one instance type, two Availability Zones, and a desired capacity of two instances\. In a situation where one Availability Zone fails, Amazon EC2 Auto Scaling automatically launches a new instance in the healthy Availability Zone to replace the one in the unhealthy Availability Zone\. Then, when the unhealthy Availability Zone returns to a healthy state later on, Amazon EC2 Auto Scaling automatically launches a new instance in this zone, which in turn terminates an instance in the unaffected zone\. 

**Note**  
When rebalancing, Amazon EC2 Auto Scaling launches new instances before terminating the old ones, so that rebalancing does not compromise the performance or availability of your application\.   
Because Amazon EC2 Auto Scaling attempts to launch new instances before terminating the old ones, being at or near the specified maximum capacity could impede or completely stop rebalancing activities\. To avoid this problem, the system can temporarily exceed the specified maximum capacity of a group by a 10 percent margin \(or by a margin of one instance, whichever is greater\) during a rebalancing activity\. The margin is extended only if the group is at or near maximum capacity and needs rebalancing, either because of user\-requested rezoning or to compensate for zone availability issues\. The extension lasts only as long as needed to rebalance the group\.