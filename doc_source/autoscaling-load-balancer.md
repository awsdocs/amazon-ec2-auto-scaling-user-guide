# Using a Load Balancer with an Auto Scaling Group<a name="autoscaling-load-balancer"></a>

The purpose of automatic scaling is to automatically increase the size of your Auto Scaling group when demand goes up and decrease it when demand goes down\. As capacity is increased or decreased, the Amazon EC2 instances being added or removed must be registered or deregistered with a load balancer\. This enables your application to automatically distribute incoming web traffic across such a dynamically changing number of instances\.

Your load balancer acts as a single point of contact for all incoming web traffic to your Auto Scaling group\. When an instance is added to your Auto Scaling group, it needs to register with the load balancer or no traffic is routed to it\. When an instance is removed from your Auto Scaling group, it must deregister from the load balancer or traffic continues to be routed to it\.

To use a load balancer with your Auto Scaling group, create the load balancer and then attach it to the group\. 

You can also use an Elastic Load Balancing health check with your instances to make sure that traffic is routed only to the healthy instances\. For more information, see [Adding Elastic Load Balancing Health Checks to an Auto Scaling Group](as-add-elb-healthcheck.md)\.

## Elastic Load Balancing Types<a name="integrations-aws-elastic-load-balancing-types"></a>

Elastic Load Balancing provides three types of load balancers that can be used with your Auto Scaling group: Classic Load Balancers, Application Load Balancers, and Network Load Balancers\. With Classic Load Balancers, instances are registered with the load balancer\. With Application Load Balancers and Network Load Balancers, instances are registered as targets with a target group\. 

When you plan to use your load balancer with an Auto Scaling group, it's not necessary to register your EC2 instances with the load balancer or target group\. When you enable Elastic Load Balancing, instances that are launched by your Auto Scaling group are automatically registered with the load balancer or target group, and instances that are terminated by your Auto Scaling group are automatically deregistered from the load balancer or target group\.

Classic Load Balancer  
Routes and load balances either at the transport layer \(TCP/SSL\), or at the application layer \(HTTP/HTTPS\)\. A Classic Load Balancer supports either EC2\-Classic or a VPC\.

Application Load Balancer  
Routes and load balances at the application layer \(HTTP/HTTPS\), and supports path\-based routing\. An Application Load Balancer can route requests to ports on one or more registered targets, such as EC2 instances, in your virtual private cloud \(VPC\)\.  
The Application Load Balancer target groups must have a target type of `instance`\. For more information, see [Target Type](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html#target-type) in the *User Guide for Application Load Balancers*\.

Network Load Balancer  
Routes and load balances at the transport layer \(TCP/UDP Layer\-4\), based on address information extracted from the TCP packet header, not from packet content\. Network Load Balancers can handle traffic bursts, retain the source IP of the client, and use a fixed IP for the life of the load balancer\.   
The Network Load Balancer target groups must have a target type of `instance`\. For more information, see [Target Type](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#target-type) in the *User Guide for Network Load Balancers*\.

To learn more about Elastic Load Balancing, see the following topics:
+ [What Is Elastic Load Balancing?](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html)
+ [What Is a Classic Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html)
+ [What Is an Application Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
+ [What Is a Network Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)

For information about integrating Amazon EC2 Auto Scaling with Elastic Load Balancing, see the following topics:

**Topics**
+ [Elastic Load Balancing Types](#integrations-aws-elastic-load-balancing-types)
+ [Attaching a Load Balancer to Your Auto Scaling Group](attach-load-balancer-asg.md)
+ [Adding Elastic Load Balancing Health Checks to an Auto Scaling Group](as-add-elb-healthcheck.md)
+ [Expanding Your Scaled and Load\-Balanced Application to an Additional Availability Zone](as-add-availability-zone.md)