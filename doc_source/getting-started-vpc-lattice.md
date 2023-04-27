# Prepare to attach a VPC Lattice target group to your Auto Scaling group<a name="getting-started-vpc-lattice"></a>

Before you attach a VPC Lattice target group to your Auto Scaling group, you must complete the following prerequisites:
+ You must have already created a VPC Lattice service network, service, listener, and target group\. For more information, see the following topics in the *VPC Lattice User Guide*:
  + [Service networks](https://docs.aws.amazon.com/vpc-lattice/latest/ug/service-networks.html)
  + [Services](https://docs.aws.amazon.com/vpc-lattice/latest/ug/services.html)
  + [Listeners](https://docs.aws.amazon.com/vpc-lattice/latest/ug/listeners.html)
  + [Target groups](https://docs.aws.amazon.com/vpc-lattice/latest/ug/target-groups.html)
+ The target group must be in the same AWS account, VPC, and Region as your Auto Scaling group\. 
+ The target group must specify a target type of `instance`\. You can't specify a target type of `ip` when using an Auto Scaling group\.
+ You must have sufficient IAM permissions to attach the target group to the Auto Scaling group\. The following example policy shows the minimum required permissions that are necessary to attach and detach target groups\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "autoscaling:AttachTrafficSources",
                  "autoscaling:DetachTrafficSources",
                  "autoscaling:DescribeTrafficSources",
                  "vpc-lattice:RegisterTargets",
                  "vpc-lattice:DeregisterTargets"
              ],
              "Resource": "*"
          }
      ]
  }
  ```
+ If the launch template for your Auto Scaling group does not contain the correct settings for VPC Lattice, such as a compatible security group, you must update the launch template\. Existing instances are not updated with the new settings when the launch template is modified\. To update existing instances, you can terminate existing instances in the Auto Scaling group\. Amazon EC2 Auto Scaling immediately starts launching new instances to replace the instances that you terminated\. Alternatively, you can start an instance refresh to replace the instances\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.
+ Before enabling the VPC Lattice health checks on your Auto Scaling group, you can configure an application\-based health check to verify that your application is responding as expected\. For more information, see [Health checks for your target groups](https://docs.aws.amazon.com/vpc-lattice/latest/ug/target-group-health-checks.html) in the *VPC Lattice User Guide*\.

## Security groups: Inbound and outbound rules<a name="vpc-lattice-security-groups"></a>

Security groups act as a firewall for associated EC2 instances, controlling both inbound and outbound traffic at the instance level\. 

**Note**  
Network configuration is sufficiently complex that we strongly recommend that you create a new security group for use with VPC Lattice\. It also makes it easier for AWS Support to help you if you need to contact them\. The following sections are based on the assumption that you follow this recommendation\.   
To learn more about creating security groups for VPC Lattice that you can use with your Auto Scaling group, see [Control traffic using security groups](https://docs.aws.amazon.com/vpc-lattice/latest/ug/security-groups.html) in the *VPC Lattice User Guide*\. To troubleshoot issues with traffic flow, consult the *VPC Lattice User Guide* for further information\.

For information about how to create a security group, see [Create a security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-security-groups.html#creating-security-group) in the *Amazon EC2 User Guide for Linux Instances* and use the following table to determine what options to select\.


| Option | Value | 
| --- | --- | 
| Name  | A name that's easy for you to remember\. | 
| Description | A description to help you identify the security group\. | 
| VPC | The same VPC as the Auto Scaling group\. | 

### Inbound rules<a name="vpc-lattice-inbound-rules"></a>

When you create a security group, it has no inbound rules\. No inbound traffic originating from clients within a VPC Lattice service network to your instance is allowed until you add inbound rules to the security group\.

To allow clients within a VPC Lattice service network to connect to instances in your Auto Scaling group, the security group for your Auto Scaling group must be set up correctly\. In this case, give it an inbound rule to allow traffic from the name of the AWS managed prefix list for VPC Lattice, instead of a specific IP address\. The VPC Lattice prefix list is a range of IP addresses used by VPC Lattice in CIDR notation\. For more information, see [Work with AWS\-managed prefix lists](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-aws-managed-prefix-lists.html) in the *Amazon VPC User Guide*\.

For information about how to add rules to a security group, see [Add rules to your security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-security-groups.html#adding-security-group-rule) in the *Amazon VPC User Guide* and use the following table to determine what options to select\.


| Option | Value | 
| --- | --- | 
|  HTTP rule  | Type: HTTP Source: com\.amazonaws\.*region*\.vpc\-lattice  | 
|  HTTPS rule  | Type: HTTPS Source: com\.amazonaws\.*region*\.vpc\-lattice  | 

The security group is stateful: it allows traffic from clients within the VPC Lattice service network to instances in your Auto Scaling group, and then sends the response back to the client it previously left\.

### Outbound rules<a name="vpc-lattice-outbound-rules"></a>

By default, a security group includes an outbound rule that allows all outbound traffic\. You can optionally remove this default rule and add an outbound rule to accommodate specific security needs\. 

## Limitations<a name="vpc-lattice-target-group-limitations"></a>
+ [Mixed instances groups](ec2-auto-scaling-mixed-instances-groups.md) are not supported\. If you try to attach a VPC Lattice target group to an Auto Scaling group that has a mixed instances policy, you receive the error message **Currently, Auto Scaling Groups with mixed instances cannot be integrated with a VPC Lattice service**\. This is because the load balancing algorithm evenly distributes load onto all available resources and assumes that instances are similar enough to handle equal loads\.