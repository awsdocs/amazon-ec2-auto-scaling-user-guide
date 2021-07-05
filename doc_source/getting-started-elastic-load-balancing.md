# Prerequisites for getting started with Elastic Load Balancing<a name="getting-started-elastic-load-balancing"></a>

Follow the procedures in the Elastic Load Balancing documentation to create the load balancer and target group\. You can skip the step for registering your Amazon EC2 instances\. Amazon EC2 Auto Scaling automatically takes care of registering instances\. For more information, see [Getting started with Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-getting-started.html) in the *Elastic Load Balancing User Guide*\. 

Alternatively, if you want to create an Application Load Balancer or Network Load Balancer, you do not need to create the load balancer and target group now\. You can create and attach a new Application Load Balancer or Network Load Balancer from the Amazon EC2 Auto Scaling console\. For more information, see [Configure an Application Load Balancer or Network Load Balancer using the Amazon EC2 Auto Scaling console](attach-load-balancer-asg.md#as-create-load-balancer-console)\. 

If necessary, create a new launch template or launch configuration for your Auto Scaling group\. Create a launch template or launch configuration with a security group that controls the traffic to and from the instances behind the load balancer\. The recommended rules depend on the type of load balancer and the types of backends that the load balancer uses\. For example, to route traffic to web servers, allow inbound HTTP access on port 80 from the load balancer\. 

You can configure the health check settings for a specific Auto Scaling group at any time\. To add Elastic Load Balancing health checks, see [Adding Elastic Load Balancing health checks to an Auto Scaling group](as-add-elb-healthcheck.md)\. Before adding these health checks, make sure that the security group for your launch template or launch configuration allows access from the load balancer on the correct port for Elastic Load Balancing to perform health checks\. 

Before deploying virtual appliances behind a Gateway Load Balancer, the launch template or launch configuration must meet the following requirements:
+ The Amazon Machine Image \(AMI\) must specify the ID of an AMI that supports GENEVE using a Gateway Load Balancer\.
+ The security groups must allow UDP traffic on port 6081\. 