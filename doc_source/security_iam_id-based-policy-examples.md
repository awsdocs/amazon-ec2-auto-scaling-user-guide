# Amazon EC2 Auto Scaling Identity\-Based Policy Examples<a name="security_iam_id-based-policy-examples"></a>

By default, IAM users and roles don't have permission to create or modify Amazon EC2 Auto Scaling resources\. They also can't perform tasks using the AWS Management Console, AWS CLI, or AWS API\. An IAM administrator must create IAM policies that give users and roles permission to perform specific API operations on the specified resources they need\. The administrator must then attach those policies to the IAM users or groups that require those permissions\.

To learn how to create an IAM identity\-based policy using these example JSON policy documents, see [Creating Policies on the JSON Tab](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-json-editor) in the *IAM User Guide*\.

**Topics**
+ [Policy Best Practices](#security_iam_service-with-iam-policy-best-practices)
+ [Predefined AWS Managed Policies](#predefined-policies-auto-scaling)
+ [Customer Managed Policy Examples](#example-policies-auto-scaling)
+ [Required Permissions to Create a Service\-Linked Role](#ec2-auto-scaling-slr-permissions)
+ [Required Permissions for the API](#ec2-auto-scaling-api-permissions)

The following shows an example of a permissions policy\.

```
{
   "Version": "2012-10-17",
   "Statement": [
   {
      "Effect": "Allow",
      "Action": "autoscaling:CreateLaunchConfiguration",
      "Resource": [
          "arn:aws:autoscaling:region:123456789012:launchConfiguration:*:launchConfigurationName/qateam-*"
      ],
      "Condition": {
          "StringEquals": { "autoscaling:InstanceType": "t2.micro" }
      }
   },
   {
      "Effect": "Allow",
      "Action": [
          "autoscaling:CreateAutoScalingGroup",
          "autoscaling:UpdateAutoScalingGroup"
      ],
      "Resource": "*",
      "Condition": {
          "StringLikeIfExists": { "autoscaling:LaunchConfigurationName": "qateam-*" }
      }
   },
   {
      "Effect": "Allow",
      "Action": [
          "autoscaling:DescribeLaunchConfigurations",
          "autoscaling:DescribeAutoScalingGroups"
      ],
      "Resource": "*"
   }]
}
```

The preceding sample policy gives users permissions to create a launch configuration if the instance type is `t2.micro`, and the name of the launch configuration starts with `qateam-`\. They can specify a launch configuration for an Auto Scaling group only if its name starts with `qateam-`\. The `Describe` actions do not support resource\-level permissions, and therefore are in a separate statement without conditions\. For more information about the elements within an IAM policy statement, see [Amazon EC2 Auto Scaling Identity\-Based Policies](control-access-using-iam.md#security_iam_service-with-iam-id-based-policies)\.

There are also additional API actions for IAM, Amazon EC2, CloudWatch, Elastic Load Balancing, and Amazon SNS that may be required depending on what access you want to provide for a user\. For example, access to the `iam:PassRole` action is required to use an [instance profile](#policy-example-pass-IAM-role), and access to the `ec2:RunInstances` action is required to use a [launch template](#policy-example-launch-template)\.

## Policy Best Practices<a name="security_iam_service-with-iam-policy-best-practices"></a>

Identity\-based policies are very powerful\. They determine whether someone can create, access, or delete Amazon EC2 Auto Scaling resources in your account\. These actions can incur costs for your AWS account\. When you create or edit identity\-based policies, follow these guidelines and recommendations:
+ **Get Started Using AWS Managed Policies** – To start using Amazon EC2 Auto Scaling quickly, use AWS managed policies to give your employees the permissions they need\. These policies are already available in your account and are maintained and updated by AWS\. For more information, see [Get Started Using Permissions With AWS Managed Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#bp-use-aws-defined-policies) in the *IAM User Guide*\.
+ **Grant Least Privilege** – When you create custom policies, grant only the permissions required to perform a task\. Start with a minimum set of permissions and grant additional permissions as necessary\. Doing so is more secure than starting with permissions that are too lenient and then trying to tighten them later\. For more information, see [Grant Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) in the *IAM User Guide*\.
+ **Enable MFA for Sensitive Operations** – For extra security, require IAM users to use multi\-factor authentication \(MFA\) to access sensitive resources or API operations\. For more information, see [Using Multi\-Factor Authentication \(MFA\) in AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html) in the *IAM User Guide*\.
+ **Use Policy Conditions for Extra Security** – To the extent that it's practical, define the conditions under which your identity\-based policies allow access to a resource\. For example, you can write conditions to specify a range of allowable IP addresses that a request must come from\. You can also write conditions to allow requests only within a specified date or time range, or to require the use of SSL or MFA\. For more information, see [IAM JSON Policy Elements: Condition](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) in the *IAM User Guide*\.

## Predefined AWS Managed Policies<a name="predefined-policies-auto-scaling"></a>

The managed policies created by AWS grant the required permissions for common use cases\. You can attach these policies to your IAM users, based on the access that they need\. Each policy provides access to all or some of the API actions for Amazon EC2 Auto Scaling\. 

The following are the AWS managed policies for Amazon EC2 Auto Scaling:
+ `AutoScalingConsoleFullAccess` — Grants full access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ `AutoScalingConsoleReadOnlyAccess` — Grants read\-only access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ `AutoScalingFullAccess` — Grants full access to Amazon EC2 Auto Scaling\.
+ `AutoScalingReadOnlyAccess` — Grants read\-only access to Amazon EC2 Auto Scaling\.

You can also use the `AmazonEC2FullAccess` policy to grant full access to all Amazon EC2 resources and related services\. 

## Customer Managed Policy Examples<a name="example-policies-auto-scaling"></a>

You can create your own custom IAM policies to allow or deny permissions for IAM users or groups to perform Amazon EC2 Auto Scaling actions\. You can attach these custom policies to the IAM users or groups that require the specified permissions\. The following examples show permissions for several common use cases\.

For information and examples on using Amazon EC2 policies, see [IAM Policies](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-policies-for-amazon-ec2.html) in the *Amazon EC2 User Guide for Linux Instances*\.

### Example: Create and Manage Launch Configurations<a name="policy-example-launch-configuration"></a>

The following policy gives users permissions to use the specified API actions to work with launch configurations\. The `Resource` element uses a wildcard to indicate that users can specify all resources with these API actions\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": [
        "autoscaling:CreateLaunchConfiguration",
        "autoscaling:DeleteLaunchConfiguration",
        "autoscaling:DescribeLaunchConfigurations"
       ],
      "Resource": "*"
   }]
}
```

For a user to work in the console, that user must have a minimum set of permissions that allow the user to describe certain Amazon EC2 resources in their AWS account\. 

The following shows an example of a permissions policy that allows users to work with launch configurations from the console\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "autoscaling:CreateLaunchConfiguration",
              "autoscaling:DeleteLaunchConfiguration",
              "autoscaling:DescribeLaunchConfigurations",
              "ec2:DescribeImages",
              "ec2:DescribeVolumes",
              "ec2:DescribeInstances",
              "ec2:DescribeInstanceAttribute",
              "ec2:DescribeKeyPairs",
              "ec2:DescribeSecurityGroups",
              "ec2:DescribeSpotInstanceRequests",
              "ec2:DescribeSpotPriceHistory",
              "ec2:DescribeVpcClassicLink",
              "ec2:DescribeVpcs",
              "ec2:DescribeSubnets"
            ],
            "Resource": "*"
        }
    ]
}
```

You can add API actions to this policy to provide more options for users, for example:
+ `iam:ListInstanceProfiles`: To choose an instance profile\.
+ `ec2:CreateSecurityGroup`: To create a new security group\.
+ `ec2:AuthorizeSecurityGroupIngress`: To add inbound rules\.
+ `ec2:CreateKeyPair`: To create a new key pair\.

### Example: Require the Latest or Default Version of a Launch Template<a name="policy-example-launch-template"></a>

The following policy gives users permissions to create and update Auto Scaling groups to use a launch template, provided that they specify either the `Latest` or `Default` version of the launch template\. 

To complete the configuration, you must give users permission to use the `ec2:RunInstances` API action\. Users without this permission receive an error that they're not authorized to use the launch template\. In this example \(second statement\), users can use all launch templates whose name starts with `qateam-`\.  

The last statement gives users access to all Auto Scaling groups and Amazon EC2 resources in your account\. Instead of listing each action explicitly as in the previous example, you can specify `ec2:Describe*` to describe all Amazon EC2 resources\. Because the `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:CreateAutoScalingGroup",
                "autoscaling:UpdateAutoScalingGroup"
            ],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "autoscaling:LaunchTemplateVersionSpecified": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": "*",
            "Condition": {
                "ArnLike": {
                   "ec2:LaunchTemplate": "arn:aws:ec2:region:123456789012:launch-template/qateam-*" 
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "autoscaling:DescribeAutoScalingGroups"
            ],
            "Resource": "*"
        }
    ]
}
```

You can add API actions to this policy to provide more options for users, for example:
+ `ec2:CreateLaunchTemplate`: To create a new launch template\.
+ `ec2:CreateLaunchTemplateVersion`: To create a new version of a launch template\.

However, be cautious about granting these permissions\. For more information, see [Controlling the Use of Launch Templates ](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#launch-template-permissions) in the *Amazon EC2 User Guide for Linux Instances*\. 

### Example: Create and Manage Auto Scaling Groups and Scaling Policies<a name="policy-example-scaling"></a>

The following policy gives users permissions to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names\. Alternatively, you can list each action explicitly instead of using wildcards\. 

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": ["autoscaling:*Scaling*"],
      "Resource": "*"
   }]
}
```

Alternatively, you can create tag\-based policies that allow actions on groups tagged with a specific key, and then apply that policy to IAM users and groups to whom you want to grant access to those Auto Scaling groups\. 

The following policy gives users permissions to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names, as long as the Auto Scaling group has the tag `purpose=webserver`\. Because the `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

```
{
   "Version": "2012-10-17",
   "Statement": [
   {
      "Effect": "Allow",
      "Action": ["autoscaling:*Scaling*"],
      "Resource": "*",
      "Condition": {
          "StringEquals": { "autoscaling:ResourceTag/purpose": "webserver" }
      }
   },
   {
      "Effect": "Allow",
      "Action": "autoscaling:Describe*Scaling*",
      "Resource": "*"
   }]
}
```

The following policy gives users permissions to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names, as long as they don't specify a minimum size less than 1 or a maximum size greater than 10\. Because the `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

```
{
   "Version": "2012-10-17",
   "Statement": [
   {
      "Effect": "Allow",
      "Action": ["autoscaling:*Scaling*"],
      "Resource": "*",
      "Condition": {
          "NumericGreaterThanEqualsIfExists": { "autoscaling:MinSize": 1 },
          "NumericLessThanEqualsIfExists": { "autoscaling:MaxSize": 10 }
      }
   },
   {
      "Effect": "Allow",
      "Action": "autoscaling:Describe*Scaling*",
      "Resource": "*"
   }]
}
```

### Example: Control Access Using Tags<a name="policy-example-tags"></a>

To give users permissions to create or tag an Auto Scaling group only if they specify certain tags, use the `aws:RequestTag` condition key\. To allow only specific tag keys, use the `aws:TagKeys` condition key with the `ForAnyValue` modifier\.

The following policy requires users to tag any Auto Scaling groups with the tags `purpose=webserver` and `cost-center=cc123`, and allows only the `purpose` and `cost-center` tags \(no other tags can be specified\)\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": [
          "autoscaling:CreateAutoScalingGroup",
          "autoscaling:CreateOrUpdateTags"
      ],
      "Resource": "*",
      "Condition": {
          "StringEquals": { 
              "aws:RequestTag/purpose": "webserver",
              "aws:RequestTag/cost-center": "cc123"
          },
          "ForAllValues:StringEquals": { "aws:TagKeys": ["purpose", "cost-center"] }
      }
   }]
}
```

The following policy requires users to specify a tag with the key `environment` in the request\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": [
          "autoscaling:CreateAutoScalingGroup",
          "autoscaling:CreateOrUpdateTags"
      ],
      "Resource": "*",
      "Condition": {
          "StringLike": { "aws:RequestTag/environment": "*" }
      }
   }]
}
```

The following policy requires users to specify at least one tag in the request, and allows only the `cost-center` and `owner` keys\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": [
          "autoscaling:CreateAutoScalingGroup",
          "autoscaling:CreateOrUpdateTags"
      ],
      "Resource": "*",
      "Condition": {
          "ForAnyValue:StringEquals": { "aws:TagKeys": ["cost-center", "owner"] }
      }
   }]
}
```

The following policy gives users access to Auto Scaling groups with the tag `allowed=true` and allows them to apply only the tag `environment=test`\. Because launch configurations do not support tags, and `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": "autoscaling:*Scaling*",
      "Resource": "*",
      "Condition": {
          "StringEquals": { "autoscaling:ResourceTag/allowed": "true" }, 
          "StringEqualsIfExists": { "aws:RequestTag/environment": "test" },
          "ForAllValues:StringEquals": { "aws:TagKeys": "environment" }
      }
   },
   {
      "Effect": "Allow",
      "Action": [ 
          "autoscaling:*LaunchConfiguration*",
          "autoscaling:Describe*"
      ],
      "Resource": "*"
   }]
}
```

### Example: Change the Capacity of Auto Scaling Groups<a name="policy-example-capacity"></a>

The following policy gives users permissions to use the `SetDesiredCapacity` action to change the capacity of any Auto Scaling group\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": "autoscaling:SetDesiredCapacity",
      "Resource": "*"
   }]
}
```

The following policy gives users permissions to use the `SetDesiredCapacity` action to change the capacity of the specified Auto Scaling groups\. Including the UUID ensures that access is granted to the specific Auto Scaling group\. The UUID for a new group is different than the UUID for a deleted group with the same name\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": "autoscaling:SetDesiredCapacity",
      "Resource": [
          "arn:aws:autoscaling:region:123456789012:autoScalingGroup:7fe02b8e-7442-4c9e-8c8e-85fa99e9b5d9:autoScalingGroupName/group-1",
          "arn:aws:autoscaling:region:123456789012:autoScalingGroup:9d8e8ea4-22e1-44c7-a14d-520f8518c2b9:autoScalingGroupName/group-2",
          "arn:aws:autoscaling:region:123456789012:autoScalingGroup:60d6b363-ae8b-467c-947f-f1d308935521:autoScalingGroupName/group-3"
      ]
   }]
}
```

The following policy gives users permissions to use the `SetDesiredCapacity` action to change the capacity of any Auto Scaling group whose name begins with `group-`\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": "autoscaling:SetDesiredCapacity",
      "Resource": [
          "arn:aws:autoscaling:region:123456789012:autoScalingGroup:*:autoScalingGroupName/group-*"
      ]
   }]
}
```

### Example: Restrict Which Scaling Policies Can Be Deleted<a name="policy-example-delete-policy"></a>

The following policy allows users to use the `autoscaling:DeletePolicy` action to delete a scaling policy but denies the action if the Auto Scaling group being acted upon has the tag `environment=production`\.

```
{
   "Version": "2012-10-17",
   "Statement": [
      {
      "Effect": "Allow",
      "Action": "autoscaling:DeletePolicy",
      "Resource": "*"
   },
   {
      "Effect": "Deny",
      "Action": "autoscaling:DeletePolicy",
      "Resource": "*",
      "Condition": {
          "StringEquals": { "autoscaling:ResourceTag/environment": "production" }
      }
   }]
}
```

### Example: Restrict Which Service\-Linked Role Can Be Passed \(Using PassRole\)<a name="policy-example-pass-role"></a>

If your users require the ability to pass custom suffix service\-linked roles to an Auto Scaling group, attach a policy to the users or roles, based on the access that they need\. We recommend that you restrict this policy to only the service\-linked roles that your users must access\. For more information about custom suffix service\-linked roles, see [Service\-Linked Roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.

The following example is helpful for facilitating the security of your customer managed CMKs if you give different service\-linked roles access to different keys\. Depending on your needs, you might have a CMK for the development team, another for the QA team, and another for the finance team\. First, create a service\-linked role that has access to the required CMK, for example, a service\-linked role named `AWSServiceRoleForAutoScaling_devteamkeyaccess`\. Then, to grant permissions to pass that service\-linked role to an Auto Scaling group, attach the policy to your IAM users as shown\. 

The policy in the following example gives users permissions to pass the `AWSServiceRoleForAutoScaling_devteamkeyaccess` role to create any Auto Scaling group whose name begins with *devteam*\. If they try to specify a different service\-linked role, they receive an error\. If they choose not to specify a service\-linked role, the default **AWSServiceRoleForAutoScaling** role is used instead\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::123456789012:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling_devteamkeyaccess",
            "Condition": {
                "StringEquals": {
                    "iam:PassedToService": [
                        "autoscaling.amazonaws.com"
                    ]
                },
                "StringLike": {
                    "iam:AssociatedResourceARN": [
                        "arn:aws:autoscaling:region:123456789012:autoScalingGroup:*:autoScalingGroupName/devteam*"
                    ]
                }
            }
        }
    ]
}
```

### Example: Restrict Which IAM Roles Can Be Passed \(Using PassRole\)<a name="policy-example-pass-IAM-role"></a>

The user that you want to create an Auto Scaling group using a launch template or launch configuration that specifies an instance profile \(a container for an IAM role\) needs a policy that includes a statement that allows the user to pass the role, like the following\. For more information, see [IAM Role for Applications That Run on Amazon EC2 Instances](us-iam-role.md)\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::123456789012:role/qateam-*",
            "Condition": {
                "StringEquals": {
                    "iam:PassedToService": [
                        "ec2.amazonaws.com"
                    ]
                }
            }
        }
    ]
}
```

## Required Permissions to Create a Service\-Linked Role<a name="ec2-auto-scaling-slr-permissions"></a>

Amazon EC2 Auto Scaling requires permissions to create a service\-linked role the first time that any user in your AWS account calls Amazon EC2 Auto Scaling API actions\. Amazon EC2 Auto Scaling creates a service\-linked role in your account, if the role does not exist already\. The service\-linked role gives permissions to Amazon EC2 Auto Scaling so that it can call other services on your behalf\. 

For automatic role creation to succeed, users must have permissions for the `iam:CreateServiceLinkedRole` action\.

```
"Action": "iam:CreateServiceLinkedRole"
```

The following shows an example of a permissions policy that allows a user to create an Amazon EC2 Auto Scaling service\-linked role for Amazon EC2 Auto Scaling\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "arn:aws:iam::*:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName":"autoscaling.amazonaws.com"
                }
            }
        }
    ]
}
```

In addition, if your users require the ability to pass custom suffix service\-linked roles to an Auto Scaling group, you must grant them permission to call the `iam:PassRole` action\. 

For more information, see [Service\-Linked Roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.

## Required Permissions for the API<a name="ec2-auto-scaling-api-permissions"></a>

When calling the following actions from the Amazon EC2 Auto Scaling API, users must have permissions from Amazon EC2 and IAM to perform certain actions\. You specify the following actions in the `Action` element of an IAM policy statement\. 

**Create an Auto Scaling group using a launch configuration**
+ `autoscaling:CreateAutoScalingGroup`
+ `iam:CreateServiceLinkedRole` \(Needed if you are using the default service\-linked role and that role does not yet exist\) 

**Create an Auto Scaling group using a launch template**
+ `autoscaling:CreateAutoScalingGroup`
+ `ec2:RunInstances` 
+ `iam:CreateServiceLinkedRole` \(Needed if you are using the default service\-linked role and that role does not yet exist\) 

**Update an Auto Scaling group that uses a launch template**
+ `autoscaling:UpdateAutoScalingGroup`
+ `ec2:RunInstances` 

**Create a launch configuration**
+ `autoscaling:CreateLaunchConfiguration`
+ `ec2:DescribeImages`
+ `ec2:DescribeInstances`
+ `ec2:DescribeInstanceAttribute`
+ `ec2:DescribeKeyPairs`
+ `ec2:DescribeSecurityGroups`
+ `ec2:DescribeSpotInstanceRequests`
+ `ec2:DescribeVpcClassicLink`