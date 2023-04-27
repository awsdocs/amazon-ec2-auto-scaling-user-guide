# Verify the attachment status of your load balancer<a name="load-balancer-status"></a>

After you attach a load balancer, it enters the `Adding` state while registering the instances in the group\. When all instances in the group are registered, it enters the `Added` state\. After at least one registered instance passes the health checks, it enters the `InService` state\. When the load balancer is in the `InService` state, Amazon EC2 Auto Scaling can terminate and replace any instances that are reported as unhealthy\. If no registered instances pass the health checks \(for example, due to a misconfigured health check\), the load balancer doesn't enter the `InService` state\. Amazon EC2 Auto Scaling doesn't terminate and replace the instances\. 

When you detach a load balancer, it enters the `Removing` state while deregistering the instances in the group\. The instances remain running after they deregister\. By default, connection draining \(deregistration delay\) is enabled for Application Load Balancers, Network Load Balancers, and Gateway Load Balancers\. If connection draining is enabled, Elastic Load Balancing waits for in\-flight requests to complete or for the maximum timeout to expire \(whichever comes first\) before it deregisters the instances\. 

You can verify the attachment status by using the AWS Command Line Interface \(AWS CLI\) or AWS SDKs\. You cannot verify the attachment status from the console\.

**To use the AWS CLI to verify the attachment status**  
The following [describe\-traffic\-sources](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-traffic-sources.html) command returns the attachment status of all traffic sources for the specified Auto Scaling group\.

```
aws autoscaling describe-traffic-sources --auto-scaling-group-name my-asg
```

The example returns the ARN of the Elastic Load Balancing target group that's attached to the Auto Scaling group, along with the attachment status of the target group in the `State` element\.

```
{
    "TrafficSources": [
        {
            "Identifier": "arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-targets/1234567890123456",
            "State": "InService",
            "Type": "elbv2"
        }
    ]
}
```