# Benefits of Amazon EC2 Auto Scaling<a name="auto-scaling-benefits"></a>

Adding Amazon EC2 Auto Scaling to your application architecture is one way to maximize the benefits of the AWS Cloud\. When you use Amazon EC2 Auto Scaling, your applications gain the following benefits:
+ Better fault tolerance\. Amazon EC2 Auto Scaling can detect when an instance is unhealthy, terminate it, and launch an instance to replace it\. You can also configure Amazon EC2 Auto Scaling to use multiple Availability Zones\. If one Availability Zone becomes unavailable, Amazon EC2 Auto Scaling can launch instances in another one to compensate\.
+ Better availability\. Amazon EC2 Auto Scaling helps ensure that your application always has the right amount of capacity to handle the current traffic demand\. 
+ Better cost management\. Amazon EC2 Auto Scaling can dynamically increase and decrease capacity as needed\. Because you pay for the EC2 instances you use, you save money by launching instances when they are needed and terminating them when they aren't\.

**Contents**
+ [Example: Covering variable demand](#autoscaling-benefits-example)
+ [Example: Web app architecture](#autoscaling-design-example)
+ [Example: Distributing instances across Availability Zones](#arch-AutoScalingMultiAZ)
  + [Instance distribution](#AutoScalingBehavior.Rebalancing)
  + [Rebalancing activities](#AutoScalingBehavior.InstanceUsage)

## Example: Covering variable demand<a name="autoscaling-benefits-example"></a>

To demonstrate some of the benefits of Amazon EC2 Auto Scaling, consider a basic web application running on AWS\. This application allows employees to search for conference rooms that they might want to use for meetings\. During the beginning and end of the week, usage of this application is minimal\. During the middle of the week, more employees are scheduling meetings, so the demand on the application increases significantly\.

The following graph shows how much of the application's capacity is used over the course of a week\.

![\[An example of the capacity demand on an application.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-diagram.png)

Traditionally, there are two ways to plan for these changes in capacity\. The first option is to add enough servers so that the application always has enough capacity to meet demand\. The downside of this option, however, is that there are days in which the application doesn't need this much capacity\. The extra capacity remains unused and, in essence, raises the cost of keeping the application running\.

![\[An example showing how buying more capacity than needed can be inefficient from a cost perspective.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-over-diagram.png)

The second option is to have enough capacity to handle the average demand on the application\. This option is less expensive, because you aren't purchasing equipment that you'll only use occasionally\. However, you risk creating a poor customer experience when the demand on the application exceeds its capacity\.

![\[An example showing how buying less capacity than needed can cause a poor customer experience.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-under-diagram.png)

By adding Amazon EC2 Auto Scaling to this application, you have a third option available\. You can add new instances to the application only when necessary, and terminate them when they're no longer needed\. Because Amazon EC2 Auto Scaling uses EC2 instances, you only have to pay for the instances you use, when you use them\. You now have a cost\-effective architecture that provides the best customer experience while minimizing expenses\.

![\[An example showing how Amazon EC2 Auto Scaling can adjust capacity as needed.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-with-as-diagram.png)

## Example: Web app architecture<a name="autoscaling-design-example"></a>

In a common web app scenario, you run multiple copies of your app simultaneously to cover the volume of your customer traffic\. These multiple copies of your application are hosted on identical EC2 instances \(cloud servers\), each handling customer requests\.

![\[An illustration of a basic Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-sample-web-architecture-diagram.png)

Amazon EC2 Auto Scaling manages the launch and termination of these EC2 instances on your behalf\. You define a set of criteria \(such as an Amazon CloudWatch alarm\) that determines when the Auto Scaling group launches or terminates EC2 instances\. Adding Auto Scaling groups to your network architecture helps make your application more highly available and fault tolerant\.

![\[An illustration of a basic Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-sample-web-architecture-diagram-with-asgs.png)

You can create as many Auto Scaling groups as you need\. For example, you can create an Auto Scaling group for each tier\.

To distribute traffic between the instances in your Auto Scaling groups, you can introduce a load balancer into your architecture\. For more information, see [Elastic Load Balancing](autoscaling-load-balancer.md)\.

## Example: Distributing instances across Availability Zones<a name="arch-AutoScalingMultiAZ"></a>

AWS resources, such as EC2 instances, are housed in highly available data centers\. To provide additional scalability and reliability, these data centers are in different physical locations\. *Regions* are large and widely dispersed geographic locations\. Each Region contains multiple distinct locations, called *Availability Zones*, which are engineered to be isolated from failures in other Availability Zones\. They provide inexpensive, low\-latency network connectivity to other Availability Zones in the same Region\. For more information, see the [Regions and endpoints](https://docs.aws.amazon.com/general/latest/gr/as.html) table in the *Amazon Web Services General Reference*\.

Amazon EC2 Auto Scaling enables you to take advantage of the safety and reliability of geographic redundancy by spanning Auto Scaling groups across multiple Availability Zones within a Region\. When one Availability Zone becomes unhealthy or unavailable, Auto Scaling launches new instances in an unaffected Availability Zone\. When the unhealthy Availability Zone returns to a healthy state, Auto Scaling automatically redistributes the application instances evenly across all of the designated Availability Zones\.

An Auto Scaling group can contain EC2 instances in one or more Availability Zones within the same Region\. However, Auto Scaling groups cannot span multiple Regions\.

For Auto Scaling groups in a VPC, the EC2 instances are launched in subnets\. You select the subnets for your EC2 instances when you create or update the Auto Scaling group\. You can select one or more subnets per Availability Zone\. For more information, see [VPCs and subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html) in the *Amazon VPC User Guide*\.

### Instance distribution<a name="AutoScalingBehavior.Rebalancing"></a>

Amazon EC2 Auto Scaling attempts to distribute instances evenly between the Availability Zones that are enabled for your Auto Scaling group\. Amazon EC2 Auto Scaling does this by attempting to launch new instances in the Availability Zone with the fewest instances\. If the attempt fails, however, Amazon EC2 Auto Scaling attempts to launch the instances in another Availability Zone until it succeeds\. For Auto Scaling groups in a VPC, if there are multiple subnets in an Availability Zone, Amazon EC2 Auto Scaling selects a subnet from the Availability Zone at random\.

![\[A typical Auto Scaling group spanning two Availability Zones.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/as-sample-web-architecture-diagram-with-asgs-and-azs.png)

### Rebalancing activities<a name="AutoScalingBehavior.InstanceUsage"></a>

After certain actions occur, your Auto Scaling group can become unbalanced between Availability Zones\. Amazon EC2 Auto Scaling compensates by rebalancing the Availability Zones\. The following actions can lead to rebalancing activity:
+ You change the Availability Zones for your group\.
+ You explicitly terminate or detach instances and the group becomes unbalanced\.
+ An Availability Zone that previously had insufficient capacity recovers and has additional capacity available\.
+ An Availability Zone that previously had a Spot price above your maximum price now has a Spot price below your maximum price\.

When rebalancing, Amazon EC2 Auto Scaling launches new instances before terminating the old ones, so that rebalancing does not compromise the performance or availability of your application\.

Because Amazon EC2 Auto Scaling attempts to launch new instances before terminating the old ones, being at or near the specified maximum capacity could impede or completely halt rebalancing activities\. To avoid this problem, the system can temporarily exceed the specified maximum capacity of a group by a 10 percent margin \(or by a 1\-instance margin, whichever is greater\) during a rebalancing activity\. The margin is extended only if the group is at or near maximum capacity and needs rebalancing, either because of user\-requested rezoning or to compensate for zone availability issues\. The extension lasts only as long as needed to rebalance the group typically a few minutes\.