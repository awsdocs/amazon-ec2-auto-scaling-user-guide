# AWS managed policies for Amazon EC2 Auto Scaling<a name="security-iam-awsmanpol"></a>

To add permissions to users, groups, and roles, it is easier to use AWS managed policies than to write policies yourself\. It takes time and expertise to [create IAM customer managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html) that provide your team with only the permissions they need\. To get started quickly, you can use our AWS managed policies\. These policies cover common use cases and are available in your AWS account\. For more information about AWS managed policies, see [AWS managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) in the *IAM User Guide*\.

AWS services maintain and update AWS managed policies\. You can't change the permissions in AWS managed policies\. Services occasionally add additional permissions to an AWS managed policy to support new features\. This type of update affects all identities \(users, groups, and roles\) where the policy is attached\. Services are most likely to update an AWS managed policy when a new feature is launched or when new operations become available\. Services do not remove permissions from an AWS managed policy, so policy updates won't break your existing permissions\.

Additionally, AWS supports managed policies for job functions that span multiple services\. For example, the `ViewOnlyAccess` AWS managed policy provides read\-only access to many AWS services and resources\. When a service launches a new feature, AWS adds read\-only permissions for new operations and resources\. For a list and descriptions of job function policies, see [AWS managed policies for job functions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html) in the *IAM User Guide*\.

## Amazon EC2 Auto Scaling managed policies<a name="predefined-policies-auto-scaling"></a>

You can attach the following managed policies to your AWS Identity and Access Management \(IAM\) identities \(users or roles\)\. Each policy provides access to all or some of the API actions for Amazon EC2 Auto Scaling\. 
+ [AutoScalingFullAccess](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AutoScalingFullAccess) — Grants full access to Amazon EC2 Auto Scaling for IAM identities that need full Amazon EC2 Auto Scaling access from the AWS CLI or SDKs, but not AWS Management Console access\. 
+ [AutoScalingReadOnlyAccess](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AutoScalingReadOnlyAccess) — Grants read\-only access to Amazon EC2 Auto Scaling for IAM identities that are making calls only to the AWS CLI or SDKs\. 
+ [AutoScalingConsoleFullAccess](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AutoScalingConsoleFullAccess) — Grants full access to Amazon EC2 Auto Scaling using the AWS Management Console\. This policy works when you are using launch configurations, but not when you are using launch templates\. 
+ [AutoScalingConsoleReadOnlyAccess](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AutoScalingConsoleReadOnlyAccess) — Grants read\-only access to Amazon EC2 Auto Scaling using the AWS Management Console\. This policy works when you are using launch configurations, but not when you are using launch templates\. 

When you are using launch templates from the console, you need to grant additional permissions specific to launch templates, which are discussed in [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\. The Amazon EC2 Auto Scaling console needs permissions for `ec2` actions so it can display information about launch templates and launch instances using launch templates\. 

## AutoScalingServiceRolePolicy AWS managed policy<a name="service-linked-role-policy"></a>

You can't attach [AutoScalingServiceRolePolicy](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/aws-service-role/AutoScalingServiceRolePolicy) to your IAM identities\. This policy is attached to a service\-linked role that allows Amazon EC2 Auto Scaling to launch and terminate instances\. For more information, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\. 

## Amazon EC2 Auto Scaling updates to AWS managed policies<a name="policy-updates"></a>

View details about updates to AWS managed policies for Amazon EC2 Auto Scaling since this service began tracking these changes\. For automatic alerts about changes to this page, subscribe to the RSS feed on the Amazon EC2 Auto Scaling Document history page\.




| Change | Description | Date | 
| --- | --- | --- | 
|  Amazon EC2 Auto Scaling adds permissions to its service\-linked role  | The `AutoScalingServiceRolePolicy` policy now grants permissions to the service to access the API actions it needs for an integration with VPC Lattice\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/security-iam-awsmanpol.html)For more information, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.  | December 6, 2022 | 
|  Amazon EC2 Auto Scaling adds permissions to its service\-linked role  | To support using an AWS Systems Manager Parameter as an alias for an AMI ID when creating a launch template, the `AutoScalingServiceRolePolicy` policy now grants permission to call the AWS Systems Manager [GetParameters](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameters.html) API action\. For more information, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\. | March 28, 2022 | 
| Amazon EC2 Auto Scaling adds permissions to its service\-linked role | To support predictive scaling, the `AutoScalingServiceRolePolicy` policy now includes permission to call the CloudWatch [GetMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html) API action\. For more information, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\. | May 19, 2021 | 
|  Amazon EC2 Auto Scaling started tracking changes  |  Amazon EC2 Auto Scaling started tracking changes for its AWS managed policies\.  | May 19, 2021 | 