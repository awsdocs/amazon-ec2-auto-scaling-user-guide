# Amazon EC2 Auto Scaling API permissions<a name="ec2-auto-scaling-api-permissions"></a>

You must grant users permission to call the Amazon EC2 Auto Scaling API actions they need, as described in [Policy actions for Amazon EC2 Auto Scaling](control-access-using-iam.md#security_iam_service-with-iam-id-based-policies-actions)\. In addition, for some Amazon EC2 Auto Scaling actions, you must grant users permission to call specific actions from other AWS APIs\.

## Required permissions from other AWS APIs<a name="required-API-permissions"></a>

In addition to Amazon EC2 Auto Scaling API permissions, users must have the following permissions from other AWS APIs to successfully perform the associated action\.

Create an Auto Scaling group \(`autoscaling:CreateAutoScalingGroup`\)  
+ `iam:CreateServiceLinkedRole` — To create the default service\-linked role if that role does not yet exist\.
+ `iam:PassRole` — To pass an IAM role to the service or to EC2 instances on launch\. Needed when a nondefault service\-linked role, an IAM role for a lifecycle hook, or a launch template that specifies an instance profile \(a container for an IAM role\) is provided\.
+ `ec2:RunInstance` — To launch instances when a launch template is provided\.
+ `ec2:CreateTags` — To tag instances and volumes on launch when a launch template with a tag specification is provided\.

Create a launch configuration \(`autoscaling:CreateLaunchConfiguration`\)  
+ `ec2:DescribeImages`
+ `ec2:DescribeInstances`
+ `ec2:DescribeInstanceAttribute`
+ `ec2:DescribeKeyPairs`
+ `ec2:DescribeSecurityGroups`
+ `ec2:DescribeSpotInstanceRequests`
+ `ec2:DescribeVpcClassicLink`
+ `iam:PassRole` — To pass an IAM role to EC2 instances on launch\. Needed when a launch configuration specifies an instance profile \(a container for an IAM role\)\.

Create a lifecycle hook \(`autoscaling:PutLifecycleHook`\)  
+ `iam:PassRole` — To pass an IAM role to the service\. Needed when an IAM role is provided\.

Attach a VPC Lattice target group \(`autoscaling:AttachTrafficSources`\)  
+ `vpc-lattice:RegisterTargets` — To automatically register instances with the target group\.

Detach a VPC Lattice target group \(`autoscaling:DetachTrafficSources`\)  
+ `vpc-lattice:DeregisterTargets` — To automatically deregister instances with the target group\.