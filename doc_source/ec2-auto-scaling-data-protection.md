# Amazon EC2 Auto Scaling and data protection<a name="ec2-auto-scaling-data-protection"></a>

The AWS [shared responsibility model](http://aws.amazon.com/compliance/shared-responsibility-model/) applies to data protection in Amazon EC2 Auto Scaling\. As described in this model, AWS is responsible for protecting the global infrastructure that runs all of the AWS Cloud\. You are responsible for maintaining control over your content that is hosted on this infrastructure\. This content includes the security configuration and management tasks for the AWS services that you use\. For more information about data privacy, see the [Data Privacy FAQ](http://aws.amazon.com/compliance/data-privacy-faq)\. For information about data protection in Europe, see the [AWS Shared Responsibility Model and GDPR](http://aws.amazon.com/blogs/security/the-aws-shared-responsibility-model-and-gdpr/) blog post on the *AWS Security Blog*\.

For data protection purposes, we recommend that you protect AWS account credentials and set up individual user accounts with AWS Identity and Access Management \(IAM\)\. That way each user is given only the permissions necessary to fulfill their job duties\. We also recommend that you secure your data in the following ways:
+ Use multi\-factor authentication \(MFA\) with each account\.
+ Use SSL/TLS to communicate with AWS resources\. We recommend TLS 1\.2 or later\.
+ Set up API and user activity logging with AWS CloudTrail\.
+ Use AWS encryption solutions, along with all default security controls within AWS services\.
+ Use advanced managed security services such as Amazon Macie, which assists in discovering and securing personal data that is stored in Amazon S3\.
+ If you require FIPS 140\-2 validated cryptographic modules when accessing AWS through a command line interface or an API, use a FIPS endpoint\. For more information about the available FIPS endpoints, see [Federal Information Processing Standard \(FIPS\) 140\-2](http://aws.amazon.com/compliance/fips/)\.

We strongly recommend that you never put sensitive identifying information, such as your customers' account numbers, into free\-form fields such as a **Name** field\. This includes when you work with Amazon EC2 Auto Scaling or other AWS services using the console, API, AWS CLI, or AWS SDKs\. Any data that you enter into Amazon EC2 Auto Scaling or other services might get picked up for inclusion in diagnostic logs\. When you provide a URL to an external server, don't include credentials information in the URL to validate your request to that server\.

## Encrypting your data using AWS KMS<a name="encryption"></a>

You can configure your Auto Scaling group to encrypt Amazon EBS volume data stored in the cloud with AWS Key Management Service customer master keys \(CMK\)\. Amazon EC2 Auto Scaling supports AWS managed and customer managed CMKs to encrypt your data\. Note that the `KmsKeyId` option to specify a customer managed CMK is not available when you use a launch configuration\. To specify a key, use a launch template instead\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\. You can also use encryption by default to enforce the encryption of the new EBS volumes and snapshot copies that you create\.

For more information about AWS KMS, see [What is AWS Key Management Service?](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)

**Related topics**
+ [Required CMK key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md)
+ [Data protection in Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/data-protection.html) in the *Amazon EC2 User Guide for Linux Instances*
+ [Encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default) in the *Amazon EC2 User Guide for Linux Instances*