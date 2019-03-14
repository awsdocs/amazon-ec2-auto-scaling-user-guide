# Required CMK Key Policy for Use with Encrypted Volumes<a name="key-policy-requirements-EBS-encryption"></a>

If you specify a customer managed Customer Master Key \(CMK\) for Amazon EBS encryption, you must give the appropriate service\-linked role access to the CMK so that Amazon EC2 Auto Scaling can launch instances on your behalf\. To do this, you must modify the CMK's key policy either when the CMK is created or at a later time\.

**Note**  
By default, Amazon EC2 Auto Scaling has access to the [AWS managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) that is used for EBS encryption\. This CMK is unique to your account and the AWS region in which it is used\. The AWS managed CMK is used for encryption unless you specify a *customer managed CMK*\. For information about creating customer managed CMKs, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\.

## Example: CMK Key Policy Sections Allowing Access to the CMK<a name="policy-example-cmk-access"></a>

The following two policy sections grant the service\-linked role named `AWSServiceRoleForAutoScaling` permissions to use the customer managed CMK\. 

```
{
   "Sid": "Allow use of the CMK",
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

When you add new sections to your CMK policy, do not change any existing sections in the policy\. For information about editing key policies, see [Using Key Policies in AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) in the *AWS Key Management Service Developer Guide*\.

**Important**  
The Principal element of the key policy is the Amazon Resource Name \(ARN\) of the service\-linked role\. When launching On\-Demand Instances, use the ARN of the service\-linked role for Amazon EC2 Auto Scaling: **AWSServiceRoleForAutoScaling**\. When launching Spot Instances, use the ARN of the **AWSServiceRoleForEC2Spot** role when using a launch configuration, or the service\-linked role for Amazon EC2 Auto Scaling when using a launch template\.

## Example: CMK Key Policy Sections Allowing Cross\-Account Access to the CMK<a name="policy-example-cmk-cross-account-access"></a>

If your customer managed CMK is in a different account than the Auto Scaling group, you must use a grant in combination with the key policy to allow access to the CMK\. For more information, see [Using Grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) in the *AWS Key Management Service Developer Guide*\.

First, add the following two sections to the CMK's key policy so that you can use the CMK with the external account \(root user\)\. 

```
{
   "Sid": "Allow use of the key",
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
   "Sid": "Allow attachment of persistent resources",
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

Then, create a grant from the external account that delegates the relevant permissions to the appropriate service\-linked role\. The Grantee Principal element of the grant is the ARN of the appropriate service\-linked role\. The Key ID is the ARN of the CMK in your account\. The following is an example [create a grant](https://docs.aws.amazon.com/cli/latest/reference/kms/create-grant.html) CLI command giving the service\-linked role named **AWSServiceRoleForAutoScaling** permissions to use the CMK: 

```
aws kms create-grant \
--region us-west-2 \
--key-id arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d \
--grantee-principal arn:aws:iam::111122223333:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling \
--operations "Encrypt" "Decrypt" "ReEncryptFrom" "ReEncryptTo" "GenerateDataKey" "GenerateDataKeyWithoutPlaintext" "DescribeKey" "CreateGrant"
```

If you have any problems configuring the cross\-account access to a customer managed CMK that is required to launch an instance with an encrypted volume, see the [troubleshooting section](ts-as-instancelaunchfailure.md#ts-as-instancelaunchfailure-12)\.