# Using a Load Balancer With an Auto Scaling Group<a name="autoscaling-load-balancer"></a>

You can automatically increase the size of your Auto Scaling group when demand goes up and decrease it when demand goes down\. As the Auto Scaling group adds and removes EC2 instances, you must ensure that the traffic for your application is distributed across all of your EC2 instances\. The Elastic Load Balancing service automatically routes incoming web traffic across such a dynamically changing number of EC2 instances\. Your load balancer acts as a single point of contact for all incoming traffic to the instances in your Auto Scaling group\. For more information, see the [Elastic Load Balancing User Guide](http://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/)\.

To use a load balancer with your Auto Scaling group, create the load balancer and then attach it to the group\.

**Topics**
+ [Attaching a Load Balancer to Your Auto Scaling Group](attach-load-balancer-asg.md)
+ [Using ELB Health Checks with Auto Scaling](as-add-elb-healthcheck.md)
+ [Expanding Your Scaled and Load\-Balanced Application to an Additional Availability Zone](as-add-availability-zone.md)