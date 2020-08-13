# Amazon EC2 Auto Scaling and data protection<a name="ec2-auto-scaling-data-protection"></a>

Amazon EC2 Auto Scaling conforms to the AWS [shared responsibility model](http://aws.amazon.com/compliance/shared-responsibility-model/), which includes regulations and guidelines for data protection\. AWS is responsible for protecting the global infrastructure that runs all of the AWS services\. AWS maintains control over data hosted on this infrastructure, including the security configuration controls for handling customer content and personal data\. AWS customers and APN Partners, acting either as data controllers or data processors, are responsible for any personal data that they put in the AWS Cloud\. 

For data protection purposes, we recommend that you protect AWS account credentials and set up individual user accounts with AWS Identity and Access Management \(IAM\), so that each user is given only the permissions necessary to fulfill their job duties\. We also recommend that you secure your data in the following ways:
+ Use multi\-factor authentication \(MFA\) with each account\.
+ Use TLS to communicate with AWS resources\.
+ Set up API and user activity logging with AWS CloudTrail\.
+ Use AWS encryption solutions, along with all default security controls within AWS services\.
+ Use advanced managed security services such as Amazon Macie, which assists in discovering and securing personal data that is stored in Amazon S3\.

We strongly recommend that you never put sensitive identifying information, such as your customers' account numbers, into free\-form fields or metadata, such as names and tags\. Any data that you enter into metadata might get picked up for inclusion in diagnostic logs\. When you provide a URL to an external server, don't include credentials information in the URL to validate your request to that server\.

For more information about data protection, see the [AWS shared responsibility model and GDPR](http://aws.amazon.com/blogs/security/the-aws-shared-responsibility-model-and-gdpr/) blog post on the *AWS Security Blog*\.

## Encrypting your data using AWS KMS<a name="encryption"></a>

You can configure your Auto Scaling group to encrypt Amazon EBS volume data stored in the cloud with AWS Key Management Service customer master keys \(CMK\)\. Amazon EC2 Auto Scaling supports AWS managed and customer managed CMKs to encrypt your data\. Note that the `KmsKeyId` option to specify a customer managed CMK is not available when you use a launch configuration\. Use a launch template instead\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md) and [Required CMK key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md)\. You can also use encryption by default to enforce the encryption of the new EBS volumes and snapshot copies that you create\.

For more information about protecting your data managed within the Amazon EC2 service, such as EBS volumes and snapshots, see [Data protection in Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/data-protection.html) and [Encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default) in the *Amazon EC2 User Guide for Linux Instances*\.

For more information about AWS KMS, see [What is AWS Key Management Service?](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)

**Related topics**
+ [Data protection in amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/data-protection.html) in the *Amazon EC2 User Guide for Linux Instances*