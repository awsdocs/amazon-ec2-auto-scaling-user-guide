# Service\-Linked Roles for Amazon EC2 Auto Scaling<a name="autoscaling-service-linked-role"></a>

Amazon EC2 Auto Scaling uses service\-linked roles for the permissions that it requires to call other AWS services on your behalf\. A service\-linked role is a unique type of IAM role that is linked directly to an AWS service\. 

Service\-linked roles provide a secure way to delegate permissions to AWS services because only the linked service can assume a service\-linked role\. For more information, see [Using Service\-Linked Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the *IAM User Guide*\. 

## Permissions Granted by AWSServiceRoleForAutoScaling<a name="service-linked-role-permissions"></a>

Amazon EC2 Auto Scaling uses the service\-linked role named `AWSServiceRoleForAutoScaling` to manage your Auto Scaling groups on your behalf\. The `AWSServiceRoleForAutoScaling` role is automatically assigned to your Auto Scaling groups unless you specify a different service\-linked role\.

The `AWSServiceRoleForAutoScaling` role is predefined with permissions to make the following calls on your behalf: 
+ `ec2:AttachClassicLinkVpc`
+ `ec2:CancelSpotInstanceRequests`
+ `ec2:CreateTags`
+ `ec2:DeleteTags`
+ `ec2:Describe*`
+ `ec2:DetachClassicLinkVpc`
+ `ec2:ModifyInstanceAttribute`
+ `ec2:RequestSpotInstances`
+ `ec2:RunInstances`
+ `ec2:TerminateInstances`
+ `elasticloadbalancing:Register*`
+ `elasticloadbalancing:Deregister*`
+ `elasticloadbalancing:Describe*`
+ `cloudwatch:DeleteAlarms`
+ `cloudwatch:DescribeAlarms`
+ `cloudwatch:PutMetricAlarm`
+ `sns:Publish`

This role trusts the `autoscaling.amazonaws.com` service to assume it\.

You must configure permissions to allow an IAM entity \(such as a user, group, or role\) to create, edit, or delete a service\-linked role\. For more information, see [Using Service\-Linked Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the *IAM User Guide*\. 

## Create the Service\-Linked Role \(Automatic\)<a name="create-service-linked-role"></a>

Under most circumstances, you don't need to manually create a service\-linked role\. Amazon EC2 Auto Scaling creates the `AWSServiceRoleForAutoScaling` service\-linked role for you the first time that you create an Auto Scaling group but do not specify a different service\-linked role\.

If you created an Auto Scaling group before March 2018, when Amazon EC2 Auto Scaling began supporting service\-linked roles, Amazon EC2 Auto Scaling created the `AWSServiceRoleForAutoScaling` role in your AWS account\. For more information, see [A New Role Appeared in My AWS Account](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_roles.html#troubleshoot_roles_new-role-appeared) in the *IAM User Guide*\.

**Important**  
Make sure that you have enabled the IAM permissions that allow an IAM entity \(such as a user, group, or role\) to create the service\-linked role\. Otherwise, the automatic creation fails\. For more information, see [Service\-Linked Role Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#service-linked-role-permissions) in the *IAM User Guide* or the information about [required user permissions](control-access-using-iam.md#required-permissions) in this guide\.

## Create the Service\-Linked Role \(Manual\)<a name="create-service-linked-role-manual"></a>

Alternatively, you can create and then specify your own service\-linked role with a custom suffix when you create your Auto Scaling group\. This can be helpful if you must give different service\-linked roles access to different customer master key \(CMK\) keys\. For more information, see [Required CMK Key Policy for Use with Encrypted Volumes](key-policy-requirements-EBS-encryption.md)\. You can also track which Auto Scaling group made an API call in your CloudTrail logs by noting the service\-linked role in use\. For the policy that restricts the access of IAM users or roles to a specific custom service\-linked role, see [Example: Restricting Which Service\-Linked Role Can Be Passed \(Using PassRole\)](control-access-using-iam.md#policy-example-pass-role)\. 

To create a service\-linked role, you can use the IAM console, AWS CLI, or IAM API\. For more information, see [Creating a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#create-service-linked-role) in the *IAM User Guide*\.

For example, use the following [https://docs.aws.amazon.com/cli/latest/reference/iam/create-service-linked-role.html](https://docs.aws.amazon.com/cli/latest/reference/iam/create-service-linked-role.html) CLI command to create a service\-linked role for Amazon EC2 Auto Scaling with the name `AWSServiceRoleForAutoScaling`\_*suffix*\. The suffix helps you identify the purpose of the role\.

```
aws iam create-service-linked-role --aws-service-name autoscaling.amazonaws.com --custom-suffix suffix
```

The output of this command includes the ARN of the service\-linked role, which you can use to grant the service\-linked role access to your CMK\. 

```
{
    "Role": {
        "RoleId": "ABCDEF0123456789ABCDEF",
        "CreateDate": "2018-08-30T21:59:18Z",
        "RoleName": "AWSServiceRoleForAutoScaling_suffix",
        "Arn": "arn:aws:iam::123456789012:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling_suffix",
        "Path": "/aws-service-role/autoscaling.amazonaws.com/",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "sts:AssumeRole"
                    ],
                    "Principal": {
                        "Service": [
                            "autoscaling.amazonaws.com"
                        ]
                    },
                    "Effect": "Allow"
                }
            ]
        }
    }
}
```

## Edit the Service\-Linked Role<a name="edit-service-linked-role"></a>

You can edit any service\-linked role that you created for use with your Auto Scaling groups\. For more information, see [Editing a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

With the `AWSServiceRoleForAutoScaling` role created by Amazon EC2 Auto Scaling, you can edit only its description and not its permissions\. 

## Delete the Service\-Linked Role<a name="delete-service-linked-role"></a>

If you no longer need to use an Auto Scaling group, we recommend that you delete its service\-linked role\. You can delete a service\-linked role only after first deleting the related AWS resources\. If a service\-linked role is used with multiple Auto Scaling groups, you must delete all Auto Scaling groups that use the service\-linked role before you can delete it\. This protects your Auto Scaling groups because you cannot inadvertently remove permissions to manage them\. For more information, see [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)\.

You can use IAM to delete the default service\-linked role or one that you've created\. For more information, see [Deleting a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#delete-service-linked-role) in the *IAM User Guide*\.

If you delete the `AWSServiceRoleForAutoScaling` service\-linked role, Amazon EC2 Auto Scaling creates the role again when you create an Auto Scaling group but do not specify a different service\-linked role\.

## Supported Regions for Amazon EC2 Auto Scaling Service\-Linked Roles<a name="slr-regions"></a>

Amazon EC2 Auto Scaling supports using service\-linked roles in all of the AWS Regions where the service is available\. For more information, see [AWS Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html)\.