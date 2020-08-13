# Required CMK key policy for use with encrypted volumes<a name="key-policy-requirements-EBS-encryption"></a>

When creating an encrypted Amazon EBS snapshot or a launch template that specifies encrypted volumes, or enabling encryption by default, you can choose one of the following AWS Key Management Service customer master keys \(CMK\) to encrypt your data: 
+ [AWS managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-managed-cmk) — An encryption key in your account that Amazon EBS creates, owns, and manages\. This is the default encryption key for a new account\. The AWS managed CMK is used for encryption unless you specify a customer managed CMK\. 
+ [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) — A custom encryption key that you create, own, and manage\. For more information, see [Creating keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\. 

  Note: Amazon EBS does not support asymmetric CMKs\. 

Amazon EC2 Auto Scaling does not need additional authorization to use the default AWS managed CMK to protect the encrypted volumes in your AWS account\. 

If you specify a customer managed CMK for Amazon EBS encryption, you must give the appropriate [service\-linked role](autoscaling-service-linked-role.md) access to the CMK so that Amazon EC2 Auto Scaling can launch instances on your behalf\. To do this, you must modify the CMK's key policy either when the CMK is created or at a later time\. 

## Configuring key policies<a name="configuring-key-policies"></a>

Use the examples on this page to configure a key policy to give Amazon EC2 Auto Scaling access to your customer managed CMK\. You must, at minimum, add two policy statements to your CMK's key policy for it to work with Amazon EC2 Auto Scaling\.
+ The first statement allows the IAM identity specified in the `Principal` element to use the CMK directly\. It includes permissions to perform the AWS KMS `Encrypt`, `Decrypt`, `ReEncrypt*`, `GenerateDataKey*`, and `DescribeKey` operations on the CMK\. 
+ The second statement allows the IAM identity specified in the `Principal` element to use grants to delegate a subset of its own permissions to AWS services that are integrated with AWS KMS or another principal\. This allows them to use the CMK to create encrypted resources on your behalf\.

When you add the new policy statements to your CMK policy, do not change any existing statements in the policy\.

For each of the following examples, arguments that must be replaced, such as a key ID or the name of a service\-linked role, are shown as *replaceable text in italics*\. In most cases, you can replace the name of the service\-linked role with the name of an Amazon EC2 Auto Scaling service\-linked role\. However, when using a launch configuration to launch Spot Instances, use the role named `AWSServiceRoleForEC2Spot`\. 

See the following resources:
+ To create a CMK with the AWS CLI, see [create\-key](https://docs.aws.amazon.com/cli/latest/reference/kms/create-key.html)\.
+ To update a CMK policy with the AWS CLI, see [put\-key\-policy](https://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html)\.
+ To find a key ID and Amazon Resource Name \(ARN\), see [Finding the key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. 
+ For information about Amazon EC2 Auto Scaling service\-linked roles, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.

**Editing Key Policies in the Console**  
The examples in the following sections show only how to add statements to a key policy, which is just one way of changing a key policy\. The easiest way to change a key policy is to use the IAM console's default view for key policies and make an IAM entity \(user or role\) one of the *key users* for the appropriate key policy\. For more information, see [Using the AWS Management Console default view](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying.html#key-policy-modifying-how-to-console-default-view) in the *AWS Key Management Service Developer Guide*\. 

**Important**  
Be cautious\. The console's default view policy statements include permissions to perform AWS KMS `Revoke` operations on the CMK\. If you give an AWS account access to a CMK in your account, and you accidentally revoke the grant that gave them this permission, external users can no longer access their encrypted data or the key that was used to encrypt their data\. 

## Example: CMK key policy sections that allow access to the CMK<a name="policy-example-cmk-access"></a>

Add the following two policy statements to the key policy of the customer managed CMK, replacing the example ARN with the ARN of the appropriate service\-linked role that is allowed access to the CMK\. In this example, the policy sections give the service\-linked role named **AWSServiceRoleForAutoScaling** permissions to use the customer managed CMK\. 

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

## Example: CMK key policy sections that allow cross\-account access to the CMK<a name="policy-example-cmk-cross-account-access"></a>

If your customer managed CMK is in a different account than the Auto Scaling group, you must use a grant in combination with the key policy to allow access to the CMK\. For more information, see [Using grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) in the *AWS Key Management Service Developer Guide*\. 

First, add the following two policy statements to the CMK's key policy, replacing the example ARN with the ARN of the external account, and specifying the account in which the key can be used\. This allows you to use IAM policies to give an IAM user or role in the specified account permission to create a grant for the CMK using the CLI command that follows\. Giving the AWS account full access to the CMK does not by itself give any IAM users or roles access to the CMK\.

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

Then, from the external account, create a grant that delegates the relevant permissions to the appropriate service\-linked role\. The `Grantee Principal` element of the grant is the ARN of the appropriate service\-linked role\. The `key-id` is the ARN of the CMK\. The following is an example [create\-a\-grant](https://docs.aws.amazon.com/cli/latest/reference/kms/create-grant.html) CLI command that gives the service\-linked role named **AWSServiceRoleForAutoScaling** in account `111122223333` permissions to use the CMK in account `444455556666`\.

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
      "Sid": "Allow creation of grant for the CMK in external account 444455556666",
      "Effect": "Allow",
      "Action": "kms:CreateGrant",
      "Resource": "arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
    }
  ]
}
```

If you have any problems configuring the cross\-account access to a customer managed CMK that is required to launch an instance with an encrypted volume, see the [troubleshooting section](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-12)\. 

For more information, see [Amazon EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Linux Instances* and the [AWS Key Management Service Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/)\. 