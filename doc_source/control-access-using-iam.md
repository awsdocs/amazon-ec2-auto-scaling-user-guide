# Controlling Access to Your Amazon EC2 Auto Scaling Resources<a name="control-access-using-iam"></a>

Amazon EC2 Auto Scaling integrates with AWS Identity and Access Management \(IAM\), a service that enables you to do the following:
+ Create users and groups under your organization's AWS account
+ Assign unique security credentials to each user under your AWS account
+ Control each user's permissions to perform tasks using AWS resources
+ Allow the users in another AWS account to share your AWS resources
+ Create roles for your AWS account and define the users or services that can assume them
+ Use existing identities for your enterprise to grant permissions to perform tasks using AWS resources

For example, you can create an IAM policy that grants the Managers group permission to use only the `DescribeAutoScalingGroups`, `DescribeLaunchConfigurations`, `DescribeScalingActivities`, and `DescribePolicies` API operations\. Users in the Managers group could then use those operations with any Auto Scaling groups and launch configurations\.

You can also create IAM policies that restrict access to a particular Auto Scaling group or launch configuration\.

For more information, see [Identity and Access Management \(IAM\)](http://aws.amazon.com/iam/) or the [IAM User Guide](http://docs.aws.amazon.com/IAM/latest/UserGuide/)\.

**Topics**
+ [Amazon EC2 Auto Scaling Actions](#policy-auto-scaling-actions)
+ [Amazon EC2 Auto Scaling Resources](#policy-auto-scaling-resources)
+ [Amazon EC2 Auto Scaling Condition Keys](#policy-auto-scaling-condition-keys)
+ [Supported Resource\-Level Permissions](#resource-level-permissions-auto-scaling)
+ [Predefined AWS Managed Policies](#predefined-policies-auto-scaling)
+ [Customer Managed Policies](#example-policies-auto-scaling)
+ [Service\-Linked Roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)
+ [Launch Auto Scaling Instances with an IAM Role](us-iam-role.md)

## Amazon EC2 Auto Scaling Actions<a name="policy-auto-scaling-actions"></a>

You can specify any and all Amazon EC2 Auto Scaling actions in an IAM policy\. Use the following prefix with the name of the action: `autoscaling:`\. For example:

```
"Action": "autoscaling:CreateAutoScalingGroup"
```

To specify multiple actions in a single statement, enclose them in square brackets and separate them with commas, as follows:

```
"Action": [
    "autoscaling:CreateAutoScalingGroup",
    "autoscaling:UpdateAutoScalingGroup"
]
```

You can also use wildcards\. For example, use `autoscaling:*` to specify all Amazon EC2 Auto Scaling actions\.

```
"Action": "autoscaling:*"
```

Use `Describe:*` to specify all actions whose names start with `Describe`\.

```
"Action": "autoscaling:Describe*"
```

For more information, see [Amazon EC2 Auto Scaling Actions](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_Operations.html) in the *Amazon EC2 Auto Scaling API Reference*\.

### Required Permissions<a name="required-permissions"></a>

When calling the following actions, you must grant IAM users permission to call the action itself, plus additional actions from Amazon EC2 or IAM\.

`CreateLaunchConfiguration`  
+ `autoscaling:CreateLaunchConfiguration`
+ `ec2:DescribeImages`
+ `ec2:DescribeInstances`
+ `ec2:DescribeInstanceAttribute`
+ `ec2:DescribeKeyPairs`
+ `ec2:DescribeSecurityGroups`
+ `ec2:DescribeSpotInstanceRequests`
+ `ec2:DescribeVpcClassicLink`

`CreateAutoScalingGroup`  
+ `autoscaling:CreateAutoScalingGroup`
+ `iam:CreateServiceLinkedRole`

## Amazon EC2 Auto Scaling Resources<a name="policy-auto-scaling-resources"></a>

For actions that support resource\-level permissions, you can control the Auto Scaling group or launch configuration that users are allowed to access\.

To specify an Auto Scaling group, you must specify its Amazon Resource Name \(ARN\) as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:autoScalingGroup:uuid:autoScalingGroupName/asg-name"
```

To specify an Auto Scaling group with `CreateAutoScalingGroup`, you must replace the UUID with `*` as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:autoScalingGroup:*:autoScalingGroupName/asg-name"
```

To specify a launch configuration, you must specify its ARN as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:launchConfiguration:uuid:launchConfigurationName/lc-name"
```

To specify a launch configuration with `CreateLaunchConfiguration`, you must replace the UUID with `*` as follows:

```
"Resource": "arn:aws:autoscaling:region:123456789012:launchConfiguration:*:launchConfigurationName/lc-name"
```

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

For actions that don't support resource\-level permissions, you must use "\*" as the resource\.

```
"Resource": "*"
```

## Amazon EC2 Auto Scaling Condition Keys<a name="policy-auto-scaling-condition-keys"></a>

When you create a policy, you can specify the conditions when the policy should take effect\. To express conditions, use predefined condition keys\. There are condition keys that are specific to Auto Scaling, plus AWS\-wide condition keys\.

The following condition keys are specific to Amazon EC2 Auto Scaling:
+ `autoscaling:ImageId`
+ `autoscaling:InstanceType`
+ `autoscaling:LaunchConfigurationName`
+ `autoscaling:LaunchTemplateVersionSpecified`
+ `autoscaling:LoadBalancerNames`
+ `autoscaling:MaxSize`
+ `autoscaling:MinSize`
+ `autoscaling:ResourceTag/key`
+ `autoscaling:SpotPrice`
+ `autoscaling:TargetGroupARNs`
+ `autoscaling:VPCZoneIdentifiers`

For a list of context keys supported by each AWS service and a list of AWS\-wide policy keys, see [AWS Service Actions and Condition Context Keys](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actionsconditions.html) and [Global and IAM Condition Context Keys](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

## Supported Resource\-Level Permissions<a name="resource-level-permissions-auto-scaling"></a>

The following table describes the Amazon EC2 Auto Scaling API actions that support resource\-level permissions, as well as the supported condition keys and resources for each action\.


| API Action | Condition Keys | Resource ARN | 
| --- | --- | --- | 
| [AttachInstances](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachInstances.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [AttachLoadBalancers](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancers.html) |   `autoscaling:LoadBalancerNames`, `autoscaling:ResourceTag/key`  | Auto Scaling group | 
| [AttachLoadBalancerTargetGroups](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancerTargetGroups.html) |   `autoscaling:ResourceTag/key`, `autoscaling:TargetGroupARNs`  | Auto Scaling group | 
| [CompleteLifecycleAction](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CompleteLifecycleAction.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [CreateAutoScalingGroup](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateAutoScalingGroup.html) |   `autoscaling:LaunchConfigurationName`, `autoscaling:LaunchTemplateVersionSpecified`, `autoscaling:LoadBalancerNames`, `autoscaling:MaxSize`, `autoscaling:MinSize`, `autoscaling:ResourceTag/key`, `autoscaling:TargetGroupARNs`, `autoscaling:VPCZoneIdentifiers`, `aws:RequestTag/key`, `aws:TagKeys`  | Auto Scaling group \(replace UUID with \*\) | 
| [CreateLaunchConfiguration](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateLaunchConfiguration.html) |   `autoscaling:ImageId`, `autoscaling:InstanceType`, `autoscaling:SpotPrice`  | Launch configuration \(replace UUID with \*\) | 
| [CreateOrUpdateTags](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateOrUpdateTags.html) |   `autoscaling:ResourceTag/key`, `aws:RequestTag/key`, `aws:TagKeys`  | Auto Scaling group | 
| [DeleteAutoScalingGroup](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DeleteAutoScalingGroup.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [DeleteLaunchConfiguration](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DeleteLaunchConfiguration.html) |  | Launch configuration | 
| [DeleteLifecycleHook](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DeleteLifecycleHook.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [DeleteNotificationConfiguration](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DeleteNotificationConfiguration.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [DeletePolicy](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DeletePolicy.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [DeleteScheduledAction](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DeleteScheduledAction.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [DeleteTags](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DeleteTags.html) |   `autoscaling:ResourceTag/key`, `aws:RequestTag/key`, `aws:TagKeys`  | Auto Scaling group | 
| [DetachInstances](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [DetachLoadBalancers](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancers.html) |   `autoscaling:LoadBalancerNames`, `autoscaling:ResourceTag/key`  | Auto Scaling group | 
| [DetachLoadBalancerTargetGroups](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancerTargetGroups.html) |   `autoscaling:ResourceTag/key`, `autoscaling:TargetGroupARNs`  | Auto Scaling group | 
| [DisableMetricsCollection](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DisableMetricsCollection.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [EnableMetricsCollection](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnableMetricsCollection.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [EnterStandby](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnterStandby.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [ExecutePolicy](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExecutePolicy.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [ExitStandby](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExitStandby.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [PutLifecycleHook](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutLifecycleHook.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [PutNotificationConfiguration](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutNotificationConfiguration.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [PutScalingPolicy](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutScalingPolicy.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [PutScheduledUpdateGroupAction](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_PutScheduledUpdateGroupAction.html) |   `autoscaling:MaxSize`, `autoscaling:MinSize`, `autoscaling:ResourceTag/key`  | Auto Scaling group | 
| [RecordLifecycleActionHeartbeat](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_RecordLifecycleActionHeartbeat.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [ResumeProcesses](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ResumeProcesses.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [SetDesiredCapacity](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetDesiredCapacity.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [SetInstanceHealth](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceHealth.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [SetInstanceProtection](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceProtection.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [SuspendProcesses](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SuspendProcesses.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [TerminateInstanceInAutoScalingGroup](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_TerminateInstanceInAutoScalingGroup.html) | autoscaling:ResourceTag/key | Auto Scaling group | 
| [UpdateAutoScalingGroup](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_UpdateAutoScalingGroup.html) |   `autoscaling:LaunchConfigurationName`, `autoscaling:LaunchTemplateVersionSpecified`, `autoscaling:MaxSize`, `autoscaling:MinSize`, `autoscaling:ResourceTag/key`, `autoscaling:VPCZoneIdentifiers`  | Auto Scaling group | 

## Predefined AWS Managed Policies<a name="predefined-policies-auto-scaling"></a>

The managed policies created by AWS grant the required permissions for common use cases\. You can attach these policies to your IAM users, based on the access that they need\. Each policy grants access to all or some of the API actions for Amazon EC2 Auto Scaling, plus additional API actions for Amazon EC2, CloudWatch, Elastic Load Balancing, IAM, and Amazon SNS\.

The following are the AWS managed policies for Amazon EC2 Auto Scaling:
+ **AutoScalingConsoleFullAccess** — Grants full access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ **AutoScalingConsoleReadOnlyAccess** — Grants read\-only access to Amazon EC2 Auto Scaling using the AWS Management Console\.
+ **AutoScalingFullAccess** — Grants full access to Amazon EC2 Auto Scaling\.
+ **AutoScalingReadOnlyAccess** — Grants read\-only access to Amazon EC2 Auto Scaling\.

## Customer Managed Policies<a name="example-policies-auto-scaling"></a>

You can create custom IAM policies that grant your IAM users permissions to perform specific actions on specific resources\. The following are example policies for Amazon EC2 Auto Scaling\.

### Example: Require a Launch Template<a name="policy-example-launch-template"></a>

When you create or update an Auto Scaling group, you can specify a launch template, a launch configuration, or an EC2 instance\. When you specify a launch template, you can configure the Auto Scaling group to use a specific launch template version, the latest version, or the default version when launching instances\.

The `autoscaling:LaunchTemplateVersionSpecified` condition key is a Boolean value that is set as follows:
+ `null` \- If no launch template is specified
+ `true` \- If a specific launch template version is used
+ `false` \- If either the latest or default launch template version is used

The following policy grants users permission to create or update an Auto Scaling group using a specific launch template version, and access the Amazon EC2 resources specified in the launch template\.

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

### Example: Create and Manage Launch Configurations<a name="policy-example-launch-configuration"></a>

The following policy grants users permission to use all Amazon EC2 Auto Scaling actions that include the string `LaunchConfiguration` in their names\. Alternatively, you can list each action explicitly instead of using wildcards\. However, the policy would not automatically apply to any new Amazon EC2 Auto Scaling actions with `LaunchConfiguration` in their names\.

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

The following policy grants users permission to create a launch configuration if the instance type is `t2.micro` and the name of the launch configuration starts with **t2micro\-**, and specify a launch configuration for an Auto Scaling group only if its name starts with **t2micro\-**\.

```
{
   "Version": "2012-10-17",
   "Statement": [
   {
      "Effect": "Allow",
      "Action": "autoscaling:CreateLaunchConfiguration",
      "Resource": [
          "arn:aws:autoscaling:us-east-2:123456789012:launchConfiguration:*:launchConfigurationName/t2micro-*"
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

The following policy grants users permission to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names\.

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

The following policy grants users permission to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names, as long as the Auto Scaling group has the tag **purpose=webserver**\. Because the `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

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

The following policy grants users permission to use all Amazon EC2 Auto Scaling actions that include the string `Scaling` in their names, as long as they don't specify a minimum size less than 1 or a maximum size greater than 10\. Because the `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

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

To grant users permission to create or tag an Auto Scaling group only if they specify specific tags, use the `aws:RequestTag` condition key\. To allow only specific tag keys, use the `aws:TagKeys` condition key with the `ForAnyValue` modifier\.

The following policy requires users to tag any Auto Scaling groups with the tags **purpose=webserver** and **cost\-center=cc123**, and allows only the **purpose** and **cost\-center ** tags \(no other tags can be specified\)\.

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

The following policy requires users to specify a tag with the key **environment** in the request\.

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

The following policy requires users to specify at least one tag in the request, and allows only the **cost\-center** and **owner** keys\.

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

The following policy grants users access to Auto Scaling groups with the tag **allowed=true** and allows them to apply only the tag **environment=test**\. Because launch configurations do not support tags and `Describe` actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\.

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

The following policy grants users permission to use the `SetDesiredCapacity` action to change the capacity of any Auto Scaling group\.

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

The following policy grants users permission to use the `SetDesiredCapacity` action to change the capacity of the specified Auto Scaling groups\. Note that including the UUID ensures that access is granted to the specific Auto Scaling group\. If you were to delete an Auto Scaling group and create a new one with the same name, the UUID for the new group would be different than the UUID for the original group\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": "autoscaling:SetDesiredCapacity",
      "Resource": [
          "arn:aws:autoscaling:us-east-2:123456789012:autoScalingGroup:7fe02b8e-7442-4c9e-8c8e-85fa99e9b5d9:autoScalingGroupName/group-1",
          "arn:aws:autoscaling:us-east-2:123456789012:autoScalingGroup:9d8e8ea4-22e1-44c7-a14d-520f8518c2b9:autoScalingGroupName/group-2",
          "arn:aws:autoscaling:us-east-2:123456789012:autoScalingGroup:60d6b363-ae8b-467c-947f-f1d308935521:autoScalingGroupName/group-3"
      ]
   }]
}
```

The following policy grants users permission to use the `SetDesiredCapacity` action to change the capacity of any Auto Scaling group whose name begins with **group\-**\.

```
{
   "Version": "2012-10-17",
   "Statement": [{
      "Effect": "Allow",
      "Action": "autoscaling:SetDesiredCapacity",
      "Resource": [
          "arn:aws:autoscaling:us-east-2:123456789012:autoScalingGroup:*:autoScalingGroupName/group-*"
      ]
   }]
}
```