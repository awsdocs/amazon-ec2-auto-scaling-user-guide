# Resilience in Amazon EC2 Auto Scaling<a name="disaster-recovery-resiliency"></a>

The AWS global infrastructure is built around AWS Regions and Availability Zones\. AWS Regions provide multiple physically separated and isolated Availability Zones, which are connected with low\-latency, high\-throughput, and highly redundant networking\. With Availability Zones, you can design and operate applications and databases that automatically fail over between zones without interruption\. Availability Zones are more highly available, fault tolerant, and scalable than traditional single or multiple data center infrastructures\. 

For more information about AWS Regions and Availability Zones, see [AWS Global Infrastructure](http://aws.amazon.com/about-aws/global-infrastructure/)\.

To benefit from the geographic redundancy of the Availability Zone design, do the following:
+ Span your Auto Scaling group across multiple Availability Zones\.
+ Maintain at least one instance in each Availability Zone\.
+ Attach a load balancer to distribute incoming traffic across the same Availability Zones\. If you use an Application Load Balancer, make sure that each EC2 instance gets a similar amount of traffic by keeping cross\-zone load balancing enabled\. This helps limit the impact of increased load on existing instances during a failover event and results in greater resiliency than without cross\-zone load balancing\.
+ Make sure that the Elastic Load Balancing health checks are configured correctly, and also that they are enabled on the Auto Scaling group\. Then, if an instance fails its health check, Elastic Load Balancing stops sending traffic to it and reroutes traffic to healthy instances, while Amazon EC2 Auto Scaling replaces the unhealthy instance\.

Amazon EC2 Auto Scaling helps support your application resiliency needs in the following ways: 
+ Checks instances for health and reachability issues\. When an instance becomes unhealthy, it automatically terminates the instance and launches a new one\. 
+ If dynamic scaling policies are in effect, automatically scales capacity according to incoming traffic\.
+ Detects issues in the reliability of the Amazon CloudWatch metrics that support scaling policies and pauses scale\-in activities when reliable metrics are not available, such as when data points are missing\.
+ Tries to maintain equivalent numbers of instances in each enabled Availability Zone as your group scales\.
+ Uses Availability Zones to maintain high availability\. When an Availability Zone becomes unhealthy, Amazon EC2 Auto Scaling does the following:
  + Launches new instances in a different Availability Zone that is enabled for your Auto Scaling group\.
  + Redistributes instances across all enabled Availability Zones when the unhealthy Availability Zone returns to a healthy state\.
+ Keeps trying to launch instances in other enabled Availability Zones if an instance fails to launch in a given Availability Zone\.
+ Automatically registers and deregisters instances with the load balancers associated with your Auto Scaling group\. This way, you don't need to register and deregister instances separately\.

For information on features to help support your Amazon EC2 data resiliency and backup needs, see [Resilience in Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/disaster-recovery-resiliency.html) in the *Amazon EC2 User Guide for Linux Instances*\.