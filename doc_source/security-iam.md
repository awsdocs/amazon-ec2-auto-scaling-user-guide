# Identity and access management for Amazon EC2 Auto Scaling<a name="security-iam"></a>

AWS Identity and Access Management \(IAM\) is an AWS service that helps an administrator securely control access to AWS resources\. IAM administrators control who can be *authenticated* \(signed in\) and *authorized* \(have permissions\) to use Amazon EC2 Auto Scaling resources\. IAM is an AWS service that you can use with no additional charge\.

To use Amazon EC2 Auto Scaling, you need an AWS account and AWS credentials\. To increase the security of your AWS account, we recommend that you use an *IAM user* to provide access credentials instead of using your AWS account credentials\. For more information, see [AWS account root user credentials vs\. IAM user credentials](https://docs.aws.amazon.com/general/latest/gr/root-vs-iam.html) in the *AWS General Reference* and [IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) in the *IAM User Guide*\. 

For an overview of IAM users and why they are important for the security of your account, see [AWS security credentials](https://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html) in the *AWS General Reference*\.

For details about working with IAM, see the [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)\. 

## Access control<a name="access-control"></a>

You can have valid credentials to authenticate your requests, but unless you have permissions you cannot create or access Amazon EC2 Auto Scaling resources\. For example, you must have permissions to create Auto Scaling groups, create launch configurations, and so on\. 

The following sections provide details on how an IAM administrator can use IAM to help secure your Amazon EC2 Auto Scaling resources, by controlling who can perform Amazon EC2 Auto Scaling actions\. 

We recommend that you read the Amazon EC2 topics first\. See [Identity and access management for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-iam.html) in the *Amazon EC2 User Guide for Linux Instances*\. After reading the topics in this section, you should have a good idea what access control permissions Amazon EC2 offers and how they can fit in with your Amazon EC2 Auto Scaling resource permissions\.

**Topics**
+ [Access control](#access-control)
+ [How Amazon EC2 Auto Scaling works with IAM](control-access-using-iam.md)
+ [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)
+ [Amazon EC2 Auto Scaling identity\-based policy examples](security_iam_id-based-policy-examples.md)
+ [Launch template support](ec2-auto-scaling-launch-template-permissions.md)
+ [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)
+ [Required CMK key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md)