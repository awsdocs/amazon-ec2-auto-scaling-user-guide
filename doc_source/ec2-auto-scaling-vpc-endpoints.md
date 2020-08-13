# Amazon EC2 Auto Scaling and interface VPC endpoints<a name="ec2-auto-scaling-vpc-endpoints"></a>

You can establish a private connection between your virtual private cloud \(VPC\) and the Amazon EC2 Auto Scaling API by creating an interface VPC endpoint\. You can use this connection to call the Amazon EC2 Auto Scaling API from your VPC without sending traffic over the internet\. The endpoint provides reliable, scalable connectivity to the Amazon EC2 Auto Scaling API\. It does this without requiring an internet gateway, NAT instance, or VPN connection\. 

Interface VPC endpoints are powered by AWS PrivateLink, a feature that enables private communication between AWS services using private IP addresses\. For more information, see [AWS PrivateLink](https://aws.amazon.com/privatelink)\.

**Note**  
You must explicitly enable each API that you want to access through an interface VPC endpoint\. For example, you might need to also configure an interface VPC endpoint for `ec2.region.amazonaws.com` to use when calling the Amazon EC2 API operations\. For more information, see [Interface VPC endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html) in the *Amazon VPC User Guide*\.

## Create an interface VPC endpoint<a name="create-vpce-as"></a>

Create an endpoint for Amazon EC2 Auto Scaling using the following service name:
+ **com\.amazonaws\.*region*\.autoscaling** — Creates an endpoint for the Amazon EC2 Auto Scaling API operations\.

For more information, see [Creating an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) in the *Amazon VPC User Guide*\. 

Enable private DNS for the endpoint to make API requests to the supported service using its default DNS hostname \(for example, `autoscaling.us-east-1.amazonaws.com`\)\. When creating an endpoint for AWS services, this setting is enabled by default\. For more information, see [Accessing a service through an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#access-service-though-endpoint) in the *Amazon VPC User Guide*\. 

You do not need to change any Amazon EC2 Auto Scaling settings\. Amazon EC2 Auto Scaling calls other AWS services using either public endpoints or private interface VPC endpoints, whichever are in use\. 

## Create a VPC endpoint policy<a name="create-vpce-policy-as"></a>

You can attach a policy to your VPC endpoint to control access to the Amazon EC2 Auto Scaling API\. The policy specifies:
+ The principal that can perform actions\.
+ The actions that can be performed\.
+ The resource on which the actions can be performed\.

The following example shows a VPC endpoint policy that denies everyone permission to delete a scaling policy through the endpoint\. The example policy also grants everyone permission to perform all other actions\.

```
{
   "Statement": [
        {
            "Action": "*",
            "Effect": "Allow",
            "Resource": "*",
            "Principal": "*"
        },
        {
            "Action": "autoscaling:DeleteScalingPolicy",
            "Effect": "Deny",
            "Resource": "*",
            "Principal": "*"
        }
    ]
}
```

For more information, see [Using VPC endpoint policies](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-access.html#vpc-endpoint-policies) in the *Amazon VPC User Guide*\.