# Maintaining a fixed number of instances in your Auto Scaling group<a name="as-maintain-instance-levels"></a>

After you have created your Auto Scaling group, the Auto Scaling group starts by launching enough EC2 instances to meet its minimum capacity \(or its desired capacity, if specified\)\. 

If a fixed number of instances are needed, this can be achieved by setting the same value for minimum, maximum, and desired capacity\. If there are no other scaling conditions attached to the Auto Scaling group, the group maintains this number of running instances even if an instance becomes unhealthy\. 

To maintain the same number of instances, Amazon EC2 Auto Scaling performs a periodic health check on running instances within an Auto Scaling group\. When it finds that an instance is unhealthy, it terminates that instance and launches a new one\. If you stop or terminate a running instance, the instance is considered to be unhealthy and is replaced\. For more information about health check replacements, see [Health checks for Auto Scaling instances](healthcheck.md)\.

To manually scale the Auto Scaling group, you can adjust the desired capacity to update the number of instances that Amazon EC2 Auto Scaling attempts to maintain\. Before you can adjust the desired capacity to a value outside of the minimum and maximum capacity range, you must update these limits\. 