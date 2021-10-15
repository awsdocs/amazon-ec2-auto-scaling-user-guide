# Required CMK key policy for use with encrypted volumes<a name="key-policy-requirements-EBS-encryption"></a>

Amazon EC2 Auto Scaling supports [service\-linked roles](autoscaling-service-linked-role.md), a new type of IAM role that gives you a more secure and transparent way to delegate permissions to Amazon Web Services\. Amazon EC2 Auto Scaling service\-linked roles are predefined by Amazon EC2 Auto Scaling and include permissions that the service requires to call other Amazon Web Services on your behalf\. The predefined permissions also include access to your AWS managed customer master keys \(CMKs\)\. However, they do not include access to your customer managed CMKs, allowing you to maintain full control over these keys\.

This topic describes how to set up the key policy that you need to launch Auto Scaling instances when you specify a customer managed CMK for Amazon EBS encryption\. 

**Note**  
Amazon EC2 Auto Scaling does not need additional authorization to use the default AWS managed CMK to protect the encrypted volumes in your account\. 

**Contents**
+ [Overview](#overview)
+ [Configuring key policies](#configuring-key-policies)
+ [Example 1: Key policy sections that allow access to the CMK](#policy-example-cmk-access)
+ [Example 2: Key policy sections that allow cross\-account access to the CMK](#policy-example-cmk-cross-account-access)
+ [Editing key policies in the AWS KMS console](#eding-key-policies-console)

## Overview<a name="overview"></a>

The following AWS KMS keys can be used for Amazon EBS encryption when Amazon EC2 Auto Scaling launches instances: 
+ [AWS managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-managed-cmk) — An encryption key in your account that Amazon EBS creates, owns, and manages\. This is the default encryption key for a new account\. The AWS managed CMK is used for encryption unless you specify a customer managed CMK\. 
+ [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) — A custom encryption key that you create, own, and manage\. For more information, see [Creating keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\. 

  Note: The key must be symmetric\. Amazon EBS does not support asymmetric CMKs\. 

You specify KMS keys when creating an encrypted snapshot or a launch template that specifies encrypted volumes, or enabling encryption by default\.

## Configuring key policies<a name="configuring-key-policies"></a>

Your KMS keys must have a key policy that allows Amazon EC2 Auto Scaling to launch instances with Amazon EBS volumes encrypted with a customer managed CMK\. 

Use the examples on this page to configure a key policy to give Amazon EC2 Auto Scaling access to your customer managed CMK\. You can modify the key policy either when the CMK is created or at a later time\. 

You must, at minimum, add two policy statements to your CMK's key policy for it to work with Amazon EC2 Auto Scaling\.
+ The first statement allows the IAM identity specified in the `Principal` element to use the CMK directly\. It includes permissions to perform the AWS KMS `Encrypt`, `Decrypt`, `ReEncrypt*`, `GenerateDataKey*`, and `DescribeKey` operations on the CMK\. 
+ The second statement allows the IAM identity specified in the `Principal` element to use grants to delegate a subset of its own permissions to Amazon Web Services that are integrated with AWS KMS or another principal\. This allows them to use the CMK to create encrypted resources on your behalf\.

When you add the new policy statements to your CMK policy, do not change any existing statements in the policy\.

For each of the following examples, arguments that must be replaced, such as a key ID or the name of a service\-linked role, are shown as *replaceable text in italics*\. In most cases, you can replace the name of the service\-linked role with the name of an Amazon EC2 Auto Scaling service\-linked role\. However, when using a launch configuration to launch Spot Instances, use the role named `AWSServiceRoleForEC2Spot`\. 

For more information, see the following resources:
+ To create a CMK with the AWS CLI, see [create\-key](https://docs.aws.amazon.com/cli/latest/reference/kms/create-key.html)\.
+ To update a CMK policy with the AWS CLI, see [put\-key\-policy](https://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html)\.
+ To find a key ID and Amazon Resource Name \(ARN\), see [Finding the key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. 
+ For information about Amazon EC2 Auto Scaling service\-linked roles, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.
+ For more information about Amazon EBS and KMS, [Amazon EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Linux Instances* and the [AWS Key Management Service Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/)\.

## Example 1: Key policy sections that allow access to the CMK<a name="policy-example-cmk-access"></a>

Add the following two policy statements to the key policy of the customer managed CMK, replacing the example ARN with the ARN of the appropriate service\-linked role that is allowed access to the CMK\. In this example, the policy sections give the service\-linked role named `AWSServiceRoleForAutoScaling` permissions to use the customer managed CMK\. 

```
{
   "Sid": "Allow service-linked role use of the CMK",
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

## Example 2: Key policy sections that allow cross\-account access to the CMK<a name="policy-example-cmk-cross-account-access"></a>

If your customer managed CMK is in a different account than the Auto Scaling group, you must use a grant in combination with the key policy to allow access to the CMK\. For more information, see [Using grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) in the *AWS Key Management Service Developer Guide*\. 

First, add the following two policy statements to the CMK's key policy, replacing the example ARN with the ARN of the external account, and specifying the account in which the key can be used\. This allows you to use IAM policies to give an IAM user or role in the specified account permission to create a grant for the CMK using the CLI command that follows\. Giving the account full access to the CMK does not by itself give any IAM users or roles access to the CMK\.

```
{
   "Sid": "Allow external account 111122223333 use of the CMK",
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

Then, from the external account, create a grant that delegates the relevant permissions to the appropriate service\-linked role\. The `Grantee Principal` element of the grant is the ARN of the appropriate service\-linked role\. The `key-id` is the ARN of the CMK\. The following is an example [create\-grant](https://docs.aws.amazon.com/cli/latest/reference/kms/create-grant.html) CLI command that gives the service\-linked role named `AWSServiceRoleForAutoScaling` in account `111122223333` permissions to use the CMK in account `444455556666`\.

```
aws kms create-grant \
  --region us-west-2 \
  --key-id arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d \
  --grantee-principal arn:aws:iam::111122223333:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling \
  --operations "Encrypt" "Decrypt" "ReEncryptFrom" "ReEncryptTo" "GenerateDataKey" "GenerateDataKeyWithoutPlaintext" "DescribeKey" "CreateGrant"
```

For this command to succeed, the user making the request must have permissions for the `CreateGrant` action\. The following example IAM policy allows an IAM user or role in account `111122223333` to create a grant for the CMK in account `444455556666`\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCreationOfGrantForTheCMKinExternalAccount444455556666",
      "Effect": "Allow",
      "Action": "kms:CreateGrant",
      "Resource": "arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
    }
  ]
}
```

After you grant these permissions, everything should work as expected\. For more information, see [this forum post](https://forums.aws.amazon.com/thread.jspa?threadID=277523)\. 

## Editing key policies in the AWS KMS console<a name="eding-key-policies-console"></a>

The examples in the previous sections show only how to add statements to a key policy, which is just one way of changing a key policy\. The easiest way to change a key policy is to use the AWS KMS console's default view for key policies and make an IAM entity \(user or role\) one of the *key users* for the appropriate key policy\. For more information, see [Using the AWS Management Console default view](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying.html#key-policy-modifying-how-to-console-default-view) in the *AWS Key Management Service Developer Guide*\. 

**Important**  
Be cautious\. The console's default view policy statements include permissions to perform AWS KMS `Revoke` operations on the CMK\. If you give an external account access to a CMK in your account, and you accidentally revoke the grant that gave them this permission, external users can no longer access their encrypted data or the key that was used to encrypt their data\. 
