# Elastic Load Balancing and Amazon EC2 Auto Scaling<a name="autoscaling-load-balancer"></a>

Elastic Load Balancing is used to automatically distribute your incoming application traffic across all the EC2 instances that you are running\. You can use Elastic Load Balancing to manage incoming requests by optimally routing traffic so that no one instance is overwhelmed\. 

To use Elastic Load Balancing with your Auto Scaling group, you set up a load balancer and then you [attach the load balancer to your Auto Scaling group](attach-load-balancer-asg.md) to register the group with the load balancer\. 

Your load balancer acts as a single point of contact for all incoming web traffic to your Auto Scaling group\. When an instance is added to your group, it needs to register with the load balancer or no traffic is routed to it\. When an instance is removed from your group, it must deregister from the load balancer or traffic continues to be routed to it\.

When you use Elastic Load Balancing with your Auto Scaling group, it's not necessary to register your EC2 instances with the load balancer\. Instances that are launched by your Auto Scaling group are automatically registered with the load balancer\. Likewise, instances that are terminated by your Auto Scaling group are automatically deregistered from the load balancer\.

After registering a load balancer with your Auto Scaling group, you can configure your Auto Scaling group to use Elastic Load Balancing metrics such as the request count per target \(or other metrics\) to scale the number of instances in the group as the demand on your instances changes\.

You can also optionally enable Amazon EC2 Auto Scaling to replace instances in your Auto Scaling group based on health checks provided by Elastic Load Balancing\.

**Topics**
+ [Elastic Load Balancing types](#integrations-aws-elastic-load-balancing-types)
+ [Attaching a load balancer](attach-load-balancer-asg.md)
+ [Adding ELB health checks](as-add-elb-healthcheck.md)
+ [Adding an Availability Zone](as-add-availability-zone.md)

## Elastic Load Balancing types<a name="integrations-aws-elastic-load-balancing-types"></a>

Elastic Load Balancing provides three types of load balancers that can be used with your Auto Scaling group: Classic Load Balancers, Application Load Balancers, and Network Load Balancers\. With Classic Load Balancers, instances are registered with the load balancer\. With Application Load Balancers and Network Load Balancers, instances are registered as targets with a target group\. 

Classic Load Balancer  
Routes and load balances either at the transport layer \(TCP/SSL\), or at the application layer \(HTTP/HTTPS\)\. A Classic Load Balancer supports either EC2\-Classic or a VPC\.

Application Load Balancer  
Routes and load balances at the application layer \(HTTP/HTTPS\), and supports path\-based routing\. An Application Load Balancer can route requests to ports on one or more registered targets, such as EC2 instances, in your virtual private cloud \(VPC\)\.  
The Application Load Balancer target groups must have a target type of `instance`\. For more information, see [Target type](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html#target-type) in the *User Guide for Application Load Balancers*\.

Network Load Balancer  
Routes and load balances at the transport layer \(TCP/UDP Layer\-4\), based on address information extracted from the TCP packet header, not from packet content\. Network Load Balancers can handle traffic bursts, retain the source IP of the client, and use a fixed IP for the life of the load balancer\.   
The Network Load Balancer target groups must have a target type of `instance`\. For more information, see [Target type](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#target-type) in the *User Guide for Network Load Balancers*\.

To learn more about Elastic Load Balancing, see the following topics:
+ [What is Elastic Load Balancing?](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html)
+ [What is a Classic Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html)
+ [What is an Application Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
+ [What is a Network Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)