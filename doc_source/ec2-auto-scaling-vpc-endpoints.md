# Amazon EC2 Auto Scaling and interface VPC endpoints<a name="ec2-auto-scaling-vpc-endpoints"></a>

You can improve the security posture of your VPC by configuring Amazon EC2 Auto Scaling to use an interface VPC endpoint\. Interface endpoints are powered by AWS PrivateLink, a technology that enables you to privately access Amazon EC2 Auto Scaling APIs by restricting all network traffic between your VPC and Amazon EC2 Auto Scaling to the AWS network\. With interface endpoints, you also don't need an internet gateway, a NAT device, or a virtual private gateway\.

You are not required to configure AWS PrivateLink, but it's recommended\. For more information about AWS PrivateLink and VPC endpoints, see [What is AWS PrivateLink?](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html) in the *AWS PrivateLink Guide*\.

**Topics**
+ [Create an interface VPC endpoint](#create-vpce-asg)
+ [Create a VPC endpoint policy](#create-vpce-policy-asg)

## Create an interface VPC endpoint<a name="create-vpce-asg"></a>

Create an endpoint for Amazon EC2 Auto Scaling using the following service name:

```
com.amazonaws.region.autoscaling
```

For more information, see [Access an AWS service using an interface VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html) in the *AWS PrivateLink Guide*\.

You do not need to change any Amazon EC2 Auto Scaling settings\. Amazon EC2 Auto Scaling calls other AWS services using either service endpoints or private interface VPC endpoints, whichever are in use\. 

## Create a VPC endpoint policy<a name="create-vpce-policy-asg"></a>

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

For more information, see [VPC endpoint policies](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-access.html#vpc-endpoint-policies) in the *AWS PrivateLink Guide*\.