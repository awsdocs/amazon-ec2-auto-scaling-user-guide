# Controlling Access to Your Amazon EC2 Auto Scaling Resources<a name="control-access-using-iam"></a>

 

Access to Amazon EC2 Auto Scaling requires credentials that AWS can use to authenticate your requests\. Those credentials must have [permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access.html) to perform Amazon EC2 Auto Scaling actions\. 

This topic provides details on how an IAM administrator can use AWS Identity and Access Management \(IAM\) to help secure your resources, by controlling who can perform Amazon EC2 Auto Scaling actions\. 

By default, a brand new IAM user has no permissions to do anything\. To grant permissions to call Amazon EC2 Auto Scaling actions, you attach an IAM policy to the IAM users or groups that require the permissions it grants\. 

For the policy that grants a user full access to Amazon EC2 Auto Scaling, see [Predefined AWS Managed Policies](#predefined-policies-auto-scaling)\. You can also create your own policies to specify allowed or denied actions and resources, as well as the conditions under which actions are allowed or denied\. For examples, see [Customer Managed Policy Examples](#example-policies-auto-scaling)\. However, we recommend that you first review the following introductory topics that explain the basic concepts and options for managing access to your Amazon EC2 Auto Scaling resources\. 

## Specifying Actions in a Policy<a name="policy-auto-scaling-actions"></a>

You can specify any and all Amazon EC2 Auto Scaling actions in an IAM policy\. For more information, see [Actions](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_Operations.html) in the *Amazon EC2 Auto Scaling API Reference*\.

To specify a single policy, you can use the following prefix with the name of the action: `autoscaling:`\.

```
"Action": "autoscaling:CreateAutoScalingGroup"
```

To specify multiple actions in a single policy, enclose them in square brackets and separate them with commas\.

```
"Action": [
    "autoscaling:CreateAutoScalingGroup",
    "autoscaling:UpdateAutoScalingGroup"
]
```

Wildcards are supported\. For example, you can use `autoscaling:*` to specify all Amazon EC2 Auto Scaling actions\. 

```
"Action": "autoscaling:*"
```

You can also use `Describe*` to specify all actions whose names start with `Describe`\.

```
"Action": "autoscaling:Describe*"
```

### Additional IAM Permissions<a name="required-permissions"></a>

Users must have additional permissions from Amazon EC2 and IAM to perform certain actions\. You specify the following actions in the `Action` element of an IAM policy statement\. 

**Create an Auto Scaling group using a launch configuration**
+ `autoscaling:CreateAutoScalingGroup`
+ `iam:CreateServiceLinkedRole`

**Create an Auto Scaling group using a launch template**
+ `autoscaling:CreateAutoScalingGroup`
+ `iam:CreateServiceLinkedRole`
+ `ec2:RunInstances` 

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

Users may require additional permissions to create or use Amazon EC2 resources, for example, to work with volumes and manage security groups from the console\. For more information, see [Example Policies for Working in the Amazon EC2 Console](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-policies-ec2-console.html) in the *Amazon EC2 User Guide for Linux Instances*\. For information about permissions for creating and updating launch templates, see [Controlling the Use of Launch Templates ](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#launch-template-permissions) in the *Amazon EC2 User Guide for Linux Instances*\.

There are also additional API actions for CloudWatch, Elastic Load Balancing, IAM, and Amazon SNS that may be required\. For example, the `iam:PassRole` action is required to use an [instance profile](us-iam-role.md)\. 

## Specifying the Resource<a name="policy-auto-scaling-resources"></a>

Access to resources can be controlled with an IAM policy\. For actions that support resource\-level permissions, you use an Amazon Resource Name \(ARN\) to identify the resource the policy applies to\. 

To specify an Auto Scaling group, you must specify its ARN as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:autoScalingGroup:uuid:autoScalingGroupName/asg-name"
```

To specify a launch configuration, you must specify its ARN as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:launchConfiguration:uuid:launchConfigurationName/lc-name"
```

To specify an Auto Scaling group with the `CreateAutoScalingGroup` action, you must replace the UUID with `*` as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:autoScalingGroup:*:autoScalingGroupName/asg-name"
```

To specify a launch configuration with the `CreateLaunchConfiguration` action, you must replace the UUID with `*` as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:launchConfiguration:*:launchConfigurationName/lc-name"
```

Alternatively, if you do not want to target a specific resource, you can use the `*` wildcard as the resource\. 

```
"Resource": "*"
```

All Amazon EC2 Auto Scaling actions can be used in an IAM policy to either grant or deny users permission to use that action\. However, not all Amazon EC2 Auto Scaling actions support resource\-level permissions, which enable you to specify the resources on which an action can be performed\. 

For actions that don't support resource\-level permissions, you must use "`*`" as the resource\. 

The following Amazon EC2 Auto Scaling actions do not support resource\-level permissions:
+ `DescribeAccountLimits`
+ `DescribeAdjustmentTypes`
+ `DescribeAutoScalingGroups`
+ `DescribeAutoScalingInstances`
+ `DescribeAutoScalingNotificationTypes`
+ `DescribeLaunchConfigurations`
+ `DescribeLifecycleHooks`
+ `DescribeLifecycleHookTypes`
+ `DescribeLoadBalancers`
+ `DescribeLoadBalancerTargetGroups`
+ `DescribeMetricCollectionTypes`
+ `DescribeNotificationConfigurations`
+ `DescribePolicies`
+ `DescribeScalingActivities`
+ `DescribeScalingProcessTypes`
+ `DescribeScheduledActions`
+ `DescribeTags`
+ `DescribeTerminationPolicyTypes`

## Specifying Conditions in a Policy<a name="policy-auto-scaling-condition-keys"></a>

For actions that support resource\-level permissions, you can control when users are allowed to use those actions based on conditions that have to be fulfilled\. 

When you grant permissions, you can use IAM policy language and predefined condition keys to specify the conditions\.

The following condition keys are specific to Amazon EC2 Auto Scaling:
+ `autoscaling:ImageId`
+ `autoscaling:InstanceType`
+ `autoscaling:InstanceTypes`
+ `autoscaling:LaunchConfigurationName`
+ `autoscaling:LaunchTemplateVersionSpecified`
+ `autoscaling:LoadBalancerNames`
+ `autoscaling:MaxSize`
+ `autoscaling:MinSize`
+ `autoscaling:ResourceTag/key`
+ `autoscaling:SpotPrice`
+ `autoscaling:TargetGroupARNs`
+ `autoscaling:VPCZoneIdentifiers`

**Important**  
For a complete list of constrainable API actions, the supported condition keys for each action, and the AWS\-wide condition keys, see [Actions, Resources, and Condition Keys for Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonec2autoscaling.html) and [AWS Global Condition Context Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

## Predefined AWS Managed Policies<a name="predefined-policies-auto-scaling"></a>

The managed policies created by AWS grant the required permissions for common use cases\. You can attach these policies to your IAM users, based on the access that they need\. Each policy grants access to all or some of the API actions for Amazon EC2 Auto Scaling\. 

The following are the AWS managed policies for Amazon EC2 Auto Scaling:
+ `AutoScalingConsoleFullAccess` — Grants full access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ `AutoScalingConsoleReadOnlyAccess` — Grants read\-only access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ `AutoScalingFullAccess` — Grants full access to Amazon EC2 Auto Scaling\.
+ `AutoScalingReadOnlyAccess` — Grants read\-only access to Amazon EC2 Auto Scaling\.

You can also use the `AmazonEC2FullAccess` policy to grant full access to all Amazon EC2 resources and related services\. 

## Customer Managed Policy Examples<a name="example-policies-auto-scaling"></a>

You can create your own custom IAM policies to allow or deny permissions for IAM users or groups to perform Amazon EC2 Auto Scaling actions\. You can attach these custom policies to the IAM users or groups that require the specified permissions\. The following examples show permissions for several common use cases\.

### Example: Restricting Which Service\-Linked Role Can Be Passed \(Using PassRole\)<a name="policy-example-pass-role"></a>

If your users require the ability to pass custom suffix service\-linked roles to an Auto Scaling group, attach a policy to the users or roles, based on the access that they need\. We recommend that you restrict this policy to only the service\-linked roles that your users must access\. For more information about custom suffix service\-linked roles, see [Service\-Linked Roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.

The following example is helpful for facilitating the security of your customer managed CMKs if you give different service\-linked roles access to different keys\. Depending on your needs, you might have a CMK for the development team, another for the QA team, and another for the finance team\. First, create a service\-linked role that has access to the required CMK, for example, a service\-linked role named `AWSServiceRoleForAutoScaling_devteamkeyaccess`\. Then, to grant permissions to pass that service\-linked role to an Auto Scaling group, attach the policy to your IAM users as shown\. 

The policy in the following example grants users permissions to pass the `AWSServiceRoleForAutoScaling_devteamkeyaccess` role to create any Auto Scaling group whose name begins with *devteam*\. If they try to specify a different service\-linked role, they receive an error\. If they choose not to specify a service\-linked role, the default **AWSServiceRoleForAutoScaling** role is used instead\.

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

### Example: Require a Launch Template<a name="policy-example-launch-template"></a>

The following policy grants IAM users permissions to create and update Auto Scaling groups, provided that they use a launch template and specify the version of the launch template that the group uses to launch instances\. Each instance uses the user\-specified launch template version during launch\. Users may access the Amazon EC2 resources specified in the launch template\. 

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
                    "autoscaling:LaunchTemplateVersionSpecified": "true"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:*"
            ],
            "Resource": "*"
        }
    ]
}
```

The `autoscaling:LaunchTemplateVersionSpecified` condition key accepts the following values:
+ `true` \- Ensures that a launch template version is specified\. 
+ `false` \- Ensures that either the `Latest` or `Default` launch template version is specified\. 
+ `null` \- Ensures that a launch template is not specified\.

The `ec2:*` grants permission to call all Amazon EC2 API actions and access all Amazon EC2 resources\. 

### Example: Create and Manage Launch Configurations<a name="policy-example-launch-configuration"></a>

The following policy grants users permissions to use all Amazon EC2 Auto Scaling actions that include the string `LaunchConfiguration` in their names\. Alternatively, you can list each action explicitly instead of using wildcards\. However, the policy does not automatically apply to any new Amazon EC2 Auto Scaling actions with `LaunchConfiguration` in their names\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": "autoscaling:*LaunchConfiguration*",
      "Resource": "*"
   }]
}
```

The following policy grants users permissions to create a launch configuration if the instance type is `t2.micro` and the name of the launch configuration starts with `t2micro-`\. They can specify a launch configuration for an Auto Scaling group only if its name starts with `t2micro-`\.

```
{
   "Version": "2012-10-17",
   "Statement": [
   {
      "Effect": "Allow",
      "Action": "autoscaling:CreateLaunchConfiguration",
      "Resource": [
          "arn:aws:autoscaling:region:123456789012:launchConfiguration:*:launchConfigurationName/t2micro-*"
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
          "StringLikeIfExists": { "autoscaling:LaunchConfigurationName": "t2micro-*" }
      }
   }]
}
```

### Example: Create and Manage Auto Scaling Groups and Scaling Policies<a name="policy-example-scaling"></a>

The following policy grants users permissions to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names\.

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

The following policy grants users permissions to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names, as long as the Auto Scaling group has the tag `purpose=webserver`\. Because the `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

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

The following policy grants users permissions to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names, as long as they don't specify a minimum size less than 1 or a maximum size greater than 10\. Because the `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

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

To grant users permissions to create or tag an Auto Scaling group only if they specify specific tags, use the `aws:RequestTag` condition key\. To allow only specific tag keys, use the `aws:TagKeys` condition key with the `ForAnyValue` modifier\.

The following policy requires users to tag any Auto Scaling groups with the tags `purpose=webserver` and `cost-center=cc123`, and allows only the `purpose` and `cost-center ` tags \(no other tags can be specified\)\.

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

The following policy grants users access to Auto Scaling groups with the tag `allowed=true` and allows them to apply only the tag `environment=test`\. Because launch configurations do not support tags, and `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

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

The following policy grants users permissions to use the `SetDesiredCapacity` action to change the capacity of any Auto Scaling group\.

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

The following policy grants users permissions to use the `SetDesiredCapacity` action to change the capacity of the specified Auto Scaling groups\. Including the UUID ensures that access is granted to the specific Auto Scaling group\. The UUID for a new group is different than the UUID for a deleted group with the same name\.

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

The following policy grants users permissions to use the `SetDesiredCapacity` action to change the capacity of any Auto Scaling group whose name begins with `group-`\.

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