# Amazon EC2 Auto Scaling and interface VPC endpoints<a name="ec2-auto-scaling-vpc-endpoints"></a>

You can improve the security posture of your VPC by configuring Amazon EC2 Auto Scaling to use an interface VPC endpoint\. Interface endpoints are powered by AWS PrivateLink, a technology that enables you to privately access Amazon EC2 Auto Scaling APIs by restricting all network traffic between your VPC and Amazon EC2 Auto Scaling to the Amazon network\. With interface endpoints, you also don't need an internet gateway, a NAT device, or a virtual private gateway\.

You are not required to configure AWS PrivateLink, but it's recommended\. For more information about AWS PrivateLink and VPC endpoints, see [Interface VPC Endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html)\.

**Topics**
+ [Create an interface VPC endpoint](#create-vpce-as)
+ [Create a VPC endpoint policy](#create-vpce-policy-as)

## Create an interface VPC endpoint<a name="create-vpce-as"></a>

You can create a VPC endpoint for the Amazon EC2 Auto Scaling service using either the Amazon VPC console or the AWS Command Line Interface \(AWS CLI\)\. 

Create an endpoint for Amazon EC2 Auto Scaling using the following service name:
+ **com\.amazonaws\.*region*\.autoscaling** — Creates an endpoint for the Amazon EC2 Auto Scaling API operations\.
+ **cn\.com\.amazonaws\.*region*\.autoscaling** — Creates an endpoint for the Amazon EC2 Auto Scaling API operations in the AWS China \(Beijing\) Region and AWS China \(Ningxia\) Region\.

For more information, see [Creating an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) in the *Amazon VPC User Guide*\. 

Enable private DNS for the endpoint to make API requests to the supported service using its default DNS hostname \(for example, `autoscaling.us-east-1.amazonaws.com`\)\. When creating an endpoint for AWS services, this setting is enabled by default\. For more information, see [Accessing a service through an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#access-service-though-endpoint) in the *Amazon VPC User Guide*\. 

You do not need to change any Amazon EC2 Auto Scaling settings\. Amazon EC2 Auto Scaling calls other AWS services using either service endpoints or private interface VPC endpoints, whichever are in use\. 

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