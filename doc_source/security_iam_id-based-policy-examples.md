# Amazon EC2 Auto Scaling identity\-based policy examples<a name="security_iam_id-based-policy-examples"></a>

By default, IAM users and roles don't have permission to create or modify Amazon EC2 Auto Scaling resources\. They also can't perform tasks using the AWS Management Console, AWS CLI, or AWS API\. An IAM administrator must create IAM policies that give users and roles permission to perform specific API operations on the specified resources they need\. The administrator must then attach those policies to the IAM users or groups that require those permissions\.

To learn how to create an IAM identity\-based policy using these example JSON policy documents, see [Creating policies on the JSON tab](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-json-editor) in the *IAM User Guide*\.

**Topics**
+ [Policy best practices](#security_iam_service-with-iam-policy-best-practices)
+ [Predefined AWS managed policies](#predefined-policies-auto-scaling)
+ [Customer managed policy examples](#example-policies-auto-scaling)
+ [Required permissions to create a service\-linked role](#ec2-auto-scaling-slr-permissions)
+ [Required permissions for the API](#ec2-auto-scaling-api-permissions)

The following shows an example of a permissions policy\.

```
{
   "Version": "2012-10-17",
   "Statement": [
   {
      "Effect": "Allow",
      "Action": [
          "autoscaling:CreateAutoScalingGroup",
          "autoscaling:UpdateAutoScalingGroup",
          "autoscaling:DeleteAutoScalingGroup"
      ],
      "Resource": "*",
      "Condition": {
          "StringEquals": { "autoscaling:ResourceTag/environment": "test" }
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

This sample policy gives users permissions to create, modify, and delete Auto Scaling groups, but only if the group uses the tag `environment=test`\. Because launch configurations do not support tags, and `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\. To learn more about the elements within an IAM policy statement, see [Amazon EC2 Auto Scaling identity\-based policies](control-access-using-iam.md#security_iam_service-with-iam-id-based-policies)\.

## Policy best practices<a name="security_iam_service-with-iam-policy-best-practices"></a>

Identity\-based policies are very powerful\. They determine whether someone can create, access, or delete Amazon EC2 Auto Scaling resources in your account\. These actions can incur costs for your AWS account\. When you create or edit identity\-based policies, follow these guidelines and recommendations:
+ **Get Started Using AWS Managed Policies** – To start using Amazon EC2 Auto Scaling quickly, use AWS managed policies to give your employees the permissions they need\. These policies are already available in your account and are maintained and updated by AWS\. For more information, see [Get Started Using Permissions With AWS Managed Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#bp-use-aws-defined-policies) in the *IAM User Guide*\.
+ **Grant Least Privilege** – When you create custom policies, grant only the permissions required to perform a task\. Start with a minimum set of permissions and grant additional permissions as necessary\. Doing so is more secure than starting with permissions that are too lenient and then trying to tighten them later\. For more information, see [Grant Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) in the *IAM User Guide*\.
+ **Enable MFA for Sensitive Operations** – For extra security, require IAM users to use multi\-factor authentication \(MFA\) to access sensitive resources or API operations\. For more information, see [Using Multi\-Factor Authentication \(MFA\) in AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html) in the *IAM User Guide*\.
+ **Use Policy Conditions for Extra Security** – To the extent that it's practical, define the conditions under which your identity\-based policies allow access to a resource\. For example, you can write conditions to specify a range of allowable IP addresses that a request must come from\. You can also write conditions to allow requests only within a specified date or time range, or to require the use of SSL or MFA\. For more information, see [IAM JSON Policy Elements: Condition](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) in the *IAM User Guide*\.

**Note**  
Some Amazon EC2 Auto Scaling API actions allow you to include specific Auto Scaling groups in your policy that can be created or modified by the action\. You can restrict the target resources for these actions by specifying individual Auto Scaling group ARNs\. As a best practice, however, we recommend that you use tag\-based policies that allow \(or deny\) actions on Auto Scaling groups with a specific tag\. 

## Predefined AWS managed policies<a name="predefined-policies-auto-scaling"></a>

The managed policies that are created by AWS grant the required permissions for common use cases\. You can attach these policies to your IAM users, based on the access that they need\. Each policy provides access to all or some of the API actions for Amazon EC2 Auto Scaling\. 

The following are the AWS managed policies for Amazon EC2 Auto Scaling:
+ `AutoScalingConsoleFullAccess` — Grants full access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ `AutoScalingConsoleReadOnlyAccess` — Grants read\-only access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ `AutoScalingFullAccess` — Grants full access to Amazon EC2 Auto Scaling\.
+ `AutoScalingReadOnlyAccess` — Grants read\-only access to Amazon EC2 Auto Scaling\.

You can also use the `AmazonEC2FullAccess` policy to grant full access to all Amazon EC2 resources and related services, including Amazon EC2 Auto Scaling, CloudWatch, and Elastic Load Balancing\. 

## Customer managed policy examples<a name="example-policies-auto-scaling"></a>

You can create your own custom IAM policies to allow or deny permissions for IAM users or groups to perform Amazon EC2 Auto Scaling actions\. You can attach these custom policies to the IAM users or groups that require the specified permissions\. The following examples show permissions for several common use cases\. 

If you are new to creating policies, we recommend that you first create an IAM user in your account and attach policies to the user\. You can use the console to verify the effects of each policy as you attach the policy to the user\. 

When creating and updating Auto Scaling groups, some actions require that certain other actions be carried out\. You can specify these other actions in the `Action` element of an IAM policy statement\. For example, there are additional API actions for Elastic Load Balancing, CloudWatch, and Amazon SNS that might be required depending on the access that you want to provide for a user\. 

**Topics**
+ [Control which tag keys and tag values can be used](#policy-example-tags)
+ [Control access to Auto Scaling resources based on tags](#policy-example-control-access-with-tags)
+ [Control the capacity limits of Auto Scaling groups](#policy-example-min-max-size)
+ [Control which IAM roles can be passed \(using PassRole\)](#policy-example-pass-IAM-role)
+ [Allow users to change the capacity of Auto Scaling groups](#policy-example-capacity)
+ [Allow users to create and use launch configurations](#policy-example-launch-configuration)
+ [Allow users to create and use launch templates](#policy-example-launch-template)

### Control which tag keys and tag values can be used<a name="policy-example-tags"></a>

You can use conditions in your IAM policies to control the tag keys and tag values that can be applied to Auto Scaling groups\. 

To give users permissions to create or tag an Auto Scaling group only if they specify certain tags, use the `aws:RequestTag` condition key\. To allow only specific tag keys, use the `aws:TagKeys` condition key with the `ForAllValues` modifier\. 

The following policy requires users to specify a tag with the key `environment` in the request\. The `"?*"` value enforces that there is some value for the tag key\. To use a wildcard, you must use the `StringLike` condition operator\. 

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
          "StringLike": { "aws:RequestTag/environment": "?*" }
      }
   }]
}
```

The following policy specifies that users can only tag Auto Scaling groups with the tags `purpose=webserver` and `cost-center=cc123`, and allows only the `purpose` and `cost-center` tags \(no other tags can be specified\)\.

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

**Note**  
For conditions, the condition key is not case\-sensitive and the condition value is case\-sensitive\. Therefore, to enforce the case\-sensitivity of a tag key, use the `aws:TagKeys` condition key, where the tag key is specified as a value in the condition\.

### Control access to Auto Scaling resources based on tags<a name="policy-example-control-access-with-tags"></a>

You can also provide tag information in your IAM policies to control access based on the tags that are attached to the Auto Scaling group by using the `autoscaling:ResourceTag` condition key\. 

#### Control access to creating and managing Auto Scaling groups and scaling policies<a name="policy-example-scaling"></a>

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

#### Control which scaling policies can be deleted<a name="policy-example-delete-policy"></a>

The following policy allows users to use the `autoscaling:DeletePolicy` action to delete a scaling policy\. However, it also denies the action if the Auto Scaling group being acted upon has the tag `environment=production`\.

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

### Control the capacity limits of Auto Scaling groups<a name="policy-example-min-max-size"></a>

Amazon EC2 Auto Scaling allows you to restrict the size of the Auto Scaling groups that can be created\. The following policy gives users permissions to create and update all Auto Scaling groups with the tag `allowed=true`, as long as they don't specify a minimum size less than 1 or a maximum size greater than 10\. 

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
          "StringEquals": { "autoscaling:ResourceTag/allowed": "true" }, 
          "NumericGreaterThanEqualsIfExists": { "autoscaling:MinSize": 1 },
          "NumericLessThanEqualsIfExists": { "autoscaling:MaxSize": 10 }
      }
   }]
}
```

### Control which IAM roles can be passed \(using PassRole\)<a name="policy-example-pass-IAM-role"></a>

If you want a user to be able to create Amazon EC2 Auto Scaling resources that specify an instance profile \(a container for an IAM role\), you must use a policy that includes a statement allowing the user to pass the role, like the following example\. By specifying the ARN, the policy grants the user the permission to pass only roles whose name begins with `qateam-`\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

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

### Allow users to change the capacity of Auto Scaling groups<a name="policy-example-capacity"></a>

The following policy gives users permissions to use the `SetDesiredCapacity` and `TerminateInstanceInAutoScalingGroup` API actions\. The `Resource` element uses a wildcard \(\*\) to indicate that users can change the capacity of any Auto Scaling group\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": [
          "autoscaling:SetDesiredCapacity",
          "autoscaling:TerminateInstanceInAutoScalingGroup"
      ],
      "Resource": "*"
   }]
}
```

If you are not using tags to control access to Auto Scaling groups, you can adjust the preceding statement to give users permissions to change the capacity of only Auto Scaling groups whose name begins with `devteam-`\. For more information about specifying the ARN value, see [Resources](control-access-using-iam.md#policy-auto-scaling-resources)\.

```
      "Resource": "arn:aws:autoscaling:region:123456789012:autoScalingGroup:*:autoScalingGroupName/devteam-*"
```

You can also specify multiple ARNs by enclosing them in a list\. Including the UUID ensures that access is granted to the specific Auto Scaling group\. The UUID for a new group is different than the UUID for a deleted group with the same name\.

```
      "Resource": [
          "arn:aws:autoscaling:region:123456789012:autoScalingGroup:7fe02b8e-7442-4c9e-8c8e-85fa99e9b5d9:autoScalingGroupName/devteam-1",
          "arn:aws:autoscaling:region:123456789012:autoScalingGroup:9d8e8ea4-22e1-44c7-a14d-520f8518c2b9:autoScalingGroupName/devteam-2",
          "arn:aws:autoscaling:region:123456789012:autoScalingGroup:60d6b363-ae8b-467c-947f-f1d308935521:autoScalingGroupName/devteam-3"
      ]
```

### Allow users to create and use launch configurations<a name="policy-example-launch-configuration"></a>

The following policy gives users permissions to create a launch configuration if the instance type is `t2.micro`, but only if the name of the launch configuration starts with `qateam-`\. For more information about specifying the ARN value, see [Resources](control-access-using-iam.md#policy-auto-scaling-resources)\. They can specify a launch configuration for an Auto Scaling group only if its name starts with `qateam-`\. 

The last part of the statement gives users permissions to describe launch configurations and to access certain Amazon EC2 resources in their AWS account\. This gives users minimum permissions to create and manage launch configurations from the Amazon EC2 Auto Scaling console\. 

```
{
   "Version": "2012-10-17",
   "Statement": [
   {
      "Effect": "Allow",
      "Action": "autoscaling:CreateLaunchConfiguration",
      "Resource": "arn:aws:autoscaling:region:123456789012:launchConfiguration:*:launchConfigurationName/qateam-*",
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
   }]
}
```

You can add API actions to this policy to provide more options for users, for example:
+ `iam:ListInstanceProfiles`: To list instance profiles\.
+ `ec2:CreateSecurityGroup`: To create a new security group\.
+ `ec2:AuthorizeSecurityGroupIngress`: To add inbound rules\.
+ `ec2:CreateKeyPair`: To create a new key pair\.

### Allow users to create and use launch templates<a name="policy-example-launch-template"></a>

For example policies, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\. 

## Required permissions to create a service\-linked role<a name="ec2-auto-scaling-slr-permissions"></a>

Amazon EC2 Auto Scaling requires permissions to create a service\-linked role the first time that any user in your AWS account calls Amazon EC2 Auto Scaling API actions\. If the service\-linked role does not exist already, Amazon EC2 Auto Scaling creates it in your account\. The service\-linked role gives permissions to Amazon EC2 Auto Scaling so that it can call other services on your behalf\. 

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

### Control which service\-linked role can be passed \(using PassRole\)<a name="policy-example-pass-role"></a>

If your users require the ability to pass custom suffix service\-linked roles to an Auto Scaling group, you must attach a policy to the users or roles, based on the access that they need\. We recommend that you restrict this policy to only the service\-linked roles that your users must access\. For more information about custom suffix service\-linked roles, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.

The following example is helpful for facilitating the security of your customer managed CMKs if you give different service\-linked roles access to different keys\. Depending on your needs, you might have a CMK for the development team, another for the QA team, and another for the finance team\. First, create a service\-linked role that has access to the required CMK, for example, a service\-linked role named **`AWSServiceRoleForAutoScaling_devteamkeyaccess`**\. Then, to grant permissions to pass that service\-linked role to an Auto Scaling group, attach the policy to your IAM users as shown\. 

The policy in this example gives users permissions to pass the **`AWSServiceRoleForAutoScaling_devteamkeyaccess`** role to create any Auto Scaling group whose name begins with `devteam-`\. If they try to specify a different service\-linked role, they receive an error\. If they choose not to specify a service\-linked role, the default **AWSServiceRoleForAutoScaling** role is used instead\.

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
                        "arn:aws:autoscaling:region:123456789012:autoScalingGroup:*:autoScalingGroupName/devteam-*"
                    ]
                }
            }
        }
    ]
}
```

## Required permissions for the API<a name="ec2-auto-scaling-api-permissions"></a>

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