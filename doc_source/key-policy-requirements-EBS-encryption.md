# Required AWS KMS key policy for use with encrypted volumes<a name="key-policy-requirements-EBS-encryption"></a>

Amazon EC2 Auto Scaling supports [service\-linked roles](autoscaling-service-linked-role.md), a new type of IAM role that gives you a more secure and transparent way to delegate permissions to Amazon Web Services\. Amazon EC2 Auto Scaling service\-linked roles are predefined by Amazon EC2 Auto Scaling and include permissions that the service requires to call other AWS services on your behalf\. The predefined permissions also include access to your AWS managed keys\. However, they do not include access to your customer managed keys, allowing you to maintain full control over these keys\.

This topic describes how to set up the key policy that you need to launch Auto Scaling instances when you specify a customer managed key for Amazon EBS encryption\. 

**Note**  
Amazon EC2 Auto Scaling does not need additional authorization to use the default AWS managed key to protect the encrypted volumes in your account\. 

**Contents**
+ [Overview](#overview)
+ [Configure key policies](#configuring-key-policies)
+ [Example 1: Key policy sections that allow access to the customer managed key](#policy-example-cmk-access)
+ [Example 2: Key policy sections that allow cross\-account access to the customer managed key](#policy-example-cmk-cross-account-access)
+ [Edit key policies in the AWS KMS console](#eding-key-policies-console)

## Overview<a name="overview"></a>

The following AWS KMS keys can be used for Amazon EBS encryption when Amazon EC2 Auto Scaling launches instances: 
+ [AWS managed key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-managed-cmk) — An encryption key in your account that Amazon EBS creates, owns, and manages\. This is the default encryption key for a new account\. The AWS managed key is used for encryption unless you specify a customer managed key\. 
+ [Customer managed key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) — A custom encryption key that you create, own, and manage\. For more information, see [Creating keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\. 

  Note: The key must be symmetric\. Amazon EBS does not support asymmetric customer managed keys\. 

You configure customer managed keys when creating encrypted snapshots or a launch template that specifies encrypted volumes, or enabling encryption by default\.

## Configure key policies<a name="configuring-key-policies"></a>

Your KMS keys must have a key policy that allows Amazon EC2 Auto Scaling to launch instances with Amazon EBS volumes encrypted with a customer managed key\. 

Use the examples on this page to configure a key policy to give Amazon EC2 Auto Scaling access to your customer managed key\. You can modify the customer managed key's key policy either when the key is created or at a later time\.

You must, at minimum, add two policy statements to your key policy for it to work with Amazon EC2 Auto Scaling\.
+ The first statement allows the IAM identity specified in the `Principal` element to use the customer managed key directly\. It includes permissions to perform the AWS KMS `Encrypt`, `Decrypt`, `ReEncrypt*`, `GenerateDataKey*`, and `DescribeKey` operations on the key\. 
+ The second statement allows the IAM identity specified in the `Principal` element to use grants to delegate a subset of its own permissions to Amazon Web Services that are integrated with AWS KMS or another principal\. This allows them to use the key to create encrypted resources on your behalf\.

When you add the new policy statements to your key policy, do not change any existing statements in the policy\.

For each of the following examples, arguments that must be replaced, such as a key ID or the name of a service\-linked role, are shown as *replaceable text in italics*\. In most cases, you can replace the name of the service\-linked role with the name of an Amazon EC2 Auto Scaling service\-linked role\. However, when using a launch configuration to launch Spot Instances, use the role named **AWSServiceRoleForEC2Spot**\. 

For more information, see the following resources:
+ To create a key with the AWS CLI, see [create\-key](https://docs.aws.amazon.com/cli/latest/reference/kms/create-key.html)\.
+ To update a key policy with the AWS CLI, see [put\-key\-policy](https://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html)\.
+ To find a key ID and Amazon Resource Name \(ARN\), see [Finding the key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. 
+ For information about Amazon EC2 Auto Scaling service\-linked roles, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.
+ For more information about Amazon EBS and KMS, [Amazon EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Linux Instances* and the [AWS Key Management Service Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/)\.

## Example 1: Key policy sections that allow access to the customer managed key<a name="policy-example-cmk-access"></a>

Add the following two policy statements to the key policy of the customer managed key, replacing the example ARN with the ARN of the appropriate service\-linked role that is allowed access to the key\. In this example, the policy sections give the service\-linked role named `AWSServiceRoleForAutoScaling` permissions to use the customer managed key\. 

```
{
   "Sid": "Allow service-linked role use of the customer managed key",
   "Effect": "Allow",
   "Principal": {
       "AWS": [
           "arn:aws:iam::123456789012:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling"
       ]
   },
   "Action": [
       "kms:Encrypt",
       "kms:Decrypt",
       "kms:ReEncrypt*",
       "kms:GenerateDataKey*",
       "kms:DescribeKey"
   ],
   "Resource": "*"
}
```

```
{
   "Sid": "Allow attachment of persistent resources",
   "Effect": "Allow",
   "Principal": {
       "AWS": [
           "arn:aws:iam::123456789012:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling"
       ]
   },
   "Action": [
       "kms:CreateGrant"
   ],
   "Resource": "*",
   "Condition": {
       "Bool": {
           "kms:GrantIsForAWSResource": true
       }
    }
}
```

## Example 2: Key policy sections that allow cross\-account access to the customer managed key<a name="policy-example-cmk-cross-account-access"></a>

If your customer managed key is in a different account than the Auto Scaling group, you must use a grant in combination with the key policy to allow access to the key\. For more information, see [Using grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) in the *AWS Key Management Service Developer Guide*\. 

First, add the following two policy statements to the customer managed key's key policy, replacing the example ARN with the ARN of the external account, and specifying the account in which the key can be used\. This allows you to use IAM policies to give an IAM user or role in the specified account permission to create a grant for the key using the CLI command that follows\. Giving the AWS account full access to the key does not by itself give any IAM users or roles access to the key\.

```
{
   "Sid": "Allow external account 111122223333 use of the customer managed key",
   "Effect": "Allow",
   "Principal": {
       "AWS": [
           "arn:aws:iam::111122223333:root"
       ]
   },
   "Action": [
       "kms:Encrypt",
       "kms:Decrypt",
       "kms:ReEncrypt*",
       "kms:GenerateDataKey*",
       "kms:DescribeKey"
   ],
   "Resource": "*"
}
```

```
{
   "Sid": "Allow attachment of persistent resources in external account 111122223333",
   "Effect": "Allow",
   "Principal": {
       "AWS": [
           "arn:aws:iam::111122223333:root"
       ]
   },
   "Action": [
       "kms:CreateGrant"
   ],
   "Resource": "*"
}
```

Then, from the external account, create a grant that delegates the relevant permissions to the appropriate service\-linked role\. The `Grantee Principal` element of the grant is the ARN of the appropriate service\-linked role\. The `key-id` is the ARN of the key\. The following is an example [create\-grant](https://docs.aws.amazon.com/cli/latest/reference/kms/create-grant.html) CLI command that gives the service\-linked role named `AWSServiceRoleForAutoScaling` in account *111122223333* permissions to use the customer managed key in account *444455556666*\.

```
aws kms create-grant \
  --region us-west-2 \
  --key-id arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d \
  --grantee-principal arn:aws:iam::111122223333:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling \
  --operations "Encrypt" "Decrypt" "ReEncryptFrom" "ReEncryptTo" "GenerateDataKey" "GenerateDataKeyWithoutPlaintext" "DescribeKey" "CreateGrant"
```

For this command to succeed, the user making the request must have permissions for the `CreateGrant` action\. The following example IAM policy allows an IAM user or role in account *111122223333* to create a grant for the customer managed key in account *444455556666*\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCreationOfGrantForTheKMSKeyinExternalAccount444455556666",
      "Effect": "Allow",
      "Action": "kms:CreateGrant",
      "Resource": "arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
    }
  ]
}
```

After you grant these permissions, everything should work as expected\.

## Edit key policies in the AWS KMS console<a name="eding-key-policies-console"></a>

The examples in the previous sections show only how to add statements to a key policy, which is just one way of changing a key policy\. The easiest way to change a key policy is to use the AWS KMS console's default view for key policies and make an IAM entity \(user or role\) one of the *key users* for the appropriate key policy\. For more information, see [Using the AWS Management Console default view](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying.html#key-policy-modifying-how-to-console-default-view) in the *AWS Key Management Service Developer Guide*\. 

**Important**  
Be cautious\. The console's default view policy statements include permissions to perform AWS KMS `Revoke` operations on the customer managed key\. If you give an AWS account access to a customer managed key in your account, and you accidentally revoke the grant that gave them this permission, external users can no longer access their encrypted data or the key that was used to encrypt their data\. 