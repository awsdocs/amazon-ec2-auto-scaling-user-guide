# Cross\-service confused deputy prevention<a name="cross-service-confused-deputy-prevention"></a>

The confused deputy problem is a security issue where an entity that doesn't have permission to perform an action can coerce a more\-privileged entity to perform the action\. 

In AWS, cross\-service impersonation can result in the confused deputy problem\. Cross\-service impersonation can occur when one service \(the *calling service*\) calls another service \(the *called service*\)\. The calling service can be manipulated to use its permissions to act on another customer's resources in a way it should not otherwise have permission to access\. 

To prevent this, AWS provides tools that help you protect your data for all services with service principals that have been given access to resources in your account\. We recommend using the [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn) and [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount) global condition context keys in trust policies for Amazon EC2 Auto Scaling service roles\. These keys limit the permissions that Amazon EC2 Auto Scaling gives another service to the resource\.

To use the `aws:SourceArn` or `aws:SourceAccount` global condition keys, set the value to the Amazon Resource Name \(ARN\) or account of the resource that Amazon EC2 Auto Scaling stores\. Whenever possible, use `aws:SourceArn`, which is more specific\. Set the value to the ARN or an ARN pattern with wildcards \(`*`\) for the unknown portions of the ARN\. If you don't know the ARN of the resource, use `aws:SourceAccount` instead\.

The following example shows how you can use the `aws:SourceArn` and `aws:SourceAccount` global condition context keys in Amazon EC2 Auto Scaling to prevent the confused deputy problem\. 

## Example: Using `aws:SourceArn` and `aws:SourceAccount` condition keys<a name="cross-service-confused-deputy-prevention-example"></a>

A role that a service assumes to perform actions on your behalf is called a [service role](control-access-using-iam.md#security_iam_service-with-iam-roles-service)\. In cases where you want to create lifecycle hooks that send notifications to anywhere other than Amazon EventBridge, you must create a service role to allow Amazon EC2 Auto Scaling to send notifications to an Amazon SNS topic or Amazon SQS queue on your behalf\. If you want only one Auto Scaling group to be associated with the cross\-service access, you can specify the trust policy of the service role as follows\.

This example trust policy uses condition statements to limit the `AssumeRole` capability on the service role to only the actions that affect the specified Auto Scaling group in the specified account\. The `aws:SourceArn` and `aws:SourceAccount` conditions are evaluated independently\. Any request to use the service role must satisfy both conditions\.

Before using this policy, replace the Region, account ID, UUID, and group name with valid values from your account\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "ConfusedDeputyPreventionExamplePolicy",
    "Effect": "Allow",
    "Principal": {
      "Service": "autoscaling.amazonaws.com"
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "ArnLike": {
        "aws:SourceArn": "arn:aws:autoscaling:region:account_id:autoScalingGroup:uuid:autoScalingGroupName/my-asg"
      },
      "StringEquals": {
        "aws:SourceAccount": "account_id"
      }
    }
  }
}
```

## Additional information<a name="cross-service-confused-deputy-prevention-additional-information"></a>

For more information, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html), [The confused deputy problem](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html), and [Modifying a role trust policy \(console\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/roles-managingrole-editing-console.html#roles-managingrole_edit-trust-policy) in the *IAM User Guide*\.