# Route traffic to your Auto Scaling group with a VPC Lattice target group<a name="ec2-auto-scaling-vpc-lattice"></a>

You can use Amazon VPC Lattice to manage the flow of traffic and API calls between your applications and services that run on separate resources, such as Auto Scaling groups or Lambda functions\. VPC Lattice is an application networking service that lets you connect, secure, and monitor all your services across multiple accounts and virtual private clouds \(VPCs\)\. To learn more about VPC Lattice, see [What is VPC Lattice?](https://docs.aws.amazon.com/vpc-lattice/latest/ug/)

To get started with VPC Lattice, first create the necessary VPC Lattice resources that enable resources in a VPC associated with a service network to connect to each other\. These resources include the services, listeners, listener rules, and target groups\. 

To associate an Auto Scaling group to a VPC Lattice service, create a target group for the service that routes requests to instances registered by instance ID, and add a listener to the service that sends requests to the target group\. Then, attach the target group to your Auto Scaling group\. Amazon EC2 Auto Scaling automatically registers the EC2 instances as targets with the target group\. Later, when Amazon EC2 Auto Scaling needs to terminate an instance, it automatically deregisters the instance from the target group before termination\.

After you attach the target group, it's the entry point for all incoming requests to your Auto Scaling group\. As the example in the following diagram shows, incoming requests can then be routed to the appropriate target group using listener rules specified for a VPC Lattice service\.

![\[VPC Lattice routes traffic to registered targets in two Auto Scaling groups using path-based routing.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/vpc-lattice-diagram-auto-scaling-groups.png)![\[VPC Lattice routes traffic to registered targets in two Auto Scaling groups using path-based routing.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)![\[VPC Lattice routes traffic to registered targets in two Auto Scaling groups using path-based routing.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/)

When traffic is routed through VPC Lattice to your Auto Scaling group, VPC Lattice balances requests among the instances in the group using round robin load balancing\. VPC Lattice also can monitor the health of its registered instances and route traffic only to healthy instances\. 

To keep your instances available for incoming requests, you can optionally add VPC Lattice health checks to your Auto Scaling group\. This way, if one of the EC2 instances fails, your Auto Scaling group automatically launches a new instance to replace it\. The behavior of the VPC Lattice health checks is similar to the behavior of the Elastic Load Balancing health checks\. The default health checks for an Auto Scaling group are EC2 health checks only\.

To learn more about VPC Lattice, see [Simplify Service\-to\-Service Connectivity, Security, and Monitoring with Amazon VPC Lattice – Now Generally Available](http://aws.amazon.com/blogs/aws/simplify-service-to-service-connectivity-security-and-monitoring-with-amazon-vpc-lattice-now-generally-available/) on the AWS Blog\.

**Topics**
+ [Prepare to attach a target group](getting-started-vpc-lattice.md)
+ [Attach a VPC Lattice target group](attach-vpc-lattice-target-group-asg.md)
+ [Verify the attachment status](verify-target-group-attachment-status.md)