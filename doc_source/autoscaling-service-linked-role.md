# Service\-linked roles for Amazon EC2 Auto Scaling<a name="autoscaling-service-linked-role"></a>

Amazon EC2 Auto Scaling uses service\-linked roles for the permissions that it requires to call other AWS services on your behalf\. A service\-linked role is a unique type of IAM role that is linked directly to an AWS service\. 

Service\-linked roles provide a secure way to delegate permissions to AWS services because only the linked service can assume a service\-linked role\. For more information, see [Using service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the *IAM User Guide*\. Service\-linked roles also enable all API calls to be visible through AWS CloudTrail\. This helps with monitoring and auditing requirements because you can track all actions that Amazon EC2 Auto Scaling performs on your behalf\. For more information, see [Logging Amazon EC2 Auto Scaling API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

The following sections describe how to create and manage Amazon EC2 Auto Scaling service\-linked roles\. Start by configuring permissions to allow an IAM entity \(such as a user, group, or role\) to create, edit, or delete a service\-linked role\. For more information, see [Using service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the *IAM User Guide*\.

## Overview<a name="autoscaling-service-linked-role-overview"></a>

There are two types of Amazon EC2 Auto Scaling service\-linked roles:
+ The default service\-linked role for your account, named **AWSServiceRoleForAutoScaling**\. This role is automatically assigned to your Auto Scaling groups unless you specify a different service\-linked role\. 
+ A service\-linked role with a custom suffix that you specify when you create the role, for example, **AWSServiceRoleForAutoScaling\_*mysuffix***\.

The permissions of a custom suffix service\-linked role are identical to those of the default service\-linked role\. In both cases, you cannot edit the roles, and you also cannot delete them if they are still in use by an Auto Scaling group\. The only difference is the role name suffix\. 

You can specify either role when you edit your AWS Key Management Service key policies to allow instances that are launched by Amazon EC2 Auto Scaling to be encrypted with your customer managed CMK\. However, if you plan to give granular access to a specific customer managed CMK, you should use a custom suffix service\-linked role\. Using a custom suffix service\-linked role provides you with:
+ More control over the CMK
+ The ability to track which Auto Scaling group made an API call in your CloudTrail logs

If you create customer managed CMKs that not all users should have access to, follow these steps to allow the use of a custom suffix service\-linked role: 

1. Create a service\-linked role with a custom suffix\. For more information, see [Create a service\-linked role \(manual\)](#create-service-linked-role-manual)\.

1. Give the service\-linked role access to a customer managed CMK\. For more information about the key policy that allows the CMK to be used by a service\-linked role, see [Required CMK key policy for use with encrypted volumes](key-policy-requirements-EBS-encryption.md)\. 

1. Give IAM users or roles access to the service\-linked role that you created\. For more information about creating the IAM policy, see [Control which service\-linked role can be passed \(using PassRole\)](security_iam_id-based-policy-examples.md#policy-example-pass-role)\. If users try to specify a service\-linked role without permission to pass that role to the service, they receive an error\.

## Permissions granted by the service\-linked role<a name="service-linked-role-permissions"></a>

Amazon EC2 Auto Scaling uses the **AWSServiceRoleForAutoScaling** service\-linked role or your custom suffix service\-linked role to make the following actions on the specified resources on your behalf: 
+ `ec2:AttachClassicLinkVpc`
+ `ec2:CancelSpotInstanceRequests`
+ `ec2:CreateFleet`
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

The role trusts the `autoscaling.amazonaws.com` service to assume it\. 

## Create a service\-linked role \(automatic\)<a name="create-service-linked-role"></a>

Amazon EC2 Auto Scaling creates the **AWSServiceRoleForAutoScaling** service\-linked role for you the first time that you create an Auto Scaling group, unless you manually create a custom suffix service\-linked role and specify it when creating the group\.

**Important**  
You must have IAM permissions to create the service\-linked role\. Otherwise, the automatic creation fails\. For more information, see [Service\-linked role permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#service-linked-role-permissions) in the *IAM User Guide* and [Required permissions to create a service\-linked role](security_iam_id-based-policy-examples.md#ec2-auto-scaling-slr-permissions) in this guide\.

Amazon EC2 Auto Scaling began supporting service\-linked roles in March 2018\. If you created an Auto Scaling group before then, Amazon EC2 Auto Scaling created the **AWSServiceRoleForAutoScaling** role in your AWS account\. For more information, see [A new role appeared in my AWS account](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_roles.html#troubleshoot_roles_new-role-appeared) in the *IAM User Guide*\.

## Create a service\-linked role \(manual\)<a name="create-service-linked-role-manual"></a>

**To create a service\-linked role \(console\)**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**, **Create role**\.

1. For **Select type of trusted entity**, choose **AWS service**\. 

1. For **Choose the service that will use this role**, choose **EC2 Auto Scaling** and the **EC2 Auto Scaling** use case\. 

1. Choose **Next: Permissions**, **Next: Tags**, and then **Next: Review**\. Note: You cannot attach tags to service\-linked roles during creation\. 

1. On the **Review** page, leave **Role name** blank to create a service\-linked role with the name **AWSServiceRoleForAutoScaling**, or enter a suffix to create a service\-linked role with the name **AWSServiceRoleForAutoScaling**\_*suffix*\.

1. \(Optional\) For **Role description**, edit the description for the service\-linked role\. 

1. Choose **Create role**\. 

**To create a service\-linked role \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/iam/create-service-linked-role.html](https://docs.aws.amazon.com/cli/latest/reference/iam/create-service-linked-role.html) CLI command to create a service\-linked role for Amazon EC2 Auto Scaling with the name **AWSServiceRoleForAutoScaling\_*suffix***\. 

```
aws iam create-service-linked-role --aws-service-name autoscaling.amazonaws.com --custom-suffix suffix
```

The output of this command includes the ARN of the service\-linked role, which you can use to give the service\-linked role access to your CMK\. 

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

For more information, see [Creating a service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#create-service-linked-role) in the *IAM User Guide*\.

## Edit the service\-linked role<a name="edit-service-linked-role"></a>

You cannot edit the service\-linked roles that are created for Amazon EC2 Auto Scaling\. After you create a service\-linked role, you cannot change the name of the role or its permissions\. However, you can edit the description of the role\. For more information, see [Editing a service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

## Delete the service\-linked role<a name="delete-service-linked-role"></a>

If you are not using an Auto Scaling group, we recommend that you delete its service\-linked role\. Deleting the role prevents you from having an entity that is not used or actively monitored and maintained\. 

You can delete a service\-linked role only after first deleting the related AWS resources\. This protects you from inadvertently revoking Amazon EC2 Auto Scaling permissions to your resources\. If a service\-linked role is used with multiple Auto Scaling groups, you must delete all Auto Scaling groups that use the service\-linked role before you can delete it\. For more information, see [Deleting your Auto Scaling infrastructure](as-process-shutdown.md)\.

You can use IAM to delete a service\-linked role\. For more information, see [Deleting a service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#delete-service-linked-role) in the *IAM User Guide*\.

If you delete the **AWSServiceRoleForAutoScaling** service\-linked role, Amazon EC2 Auto Scaling creates the role again when you create an Auto Scaling group and do not specify a different service\-linked role\.

## Supported regions for Amazon EC2 Auto Scaling service\-linked roles<a name="slr-regions"></a>

Amazon EC2 Auto Scaling supports using service\-linked roles in all of the AWS Regions where the service is available\.