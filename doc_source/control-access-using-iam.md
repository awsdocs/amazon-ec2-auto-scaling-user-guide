# How Amazon EC2 Auto Scaling works with IAM<a name="control-access-using-iam"></a>

Before you use IAM to manage access to Amazon EC2 Auto Scaling, you should understand what IAM features are available to use with Amazon EC2 Auto Scaling\. To get a high\-level view of how Amazon EC2 Auto Scaling and other Amazon Web Services work with IAM, see [Amazon Web Services that work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) in the *IAM User Guide*\.

**Note**  
Specific IAM permissions and an Amazon EC2 Auto Scaling service\-linked role are required so that users can configure Auto Scaling groups\. 

**Contents**
+ [Amazon EC2 Auto Scaling identity\-based policies](#security_iam_service-with-iam-id-based-policies)
  + [Actions](#security_iam_service-with-iam-id-based-policies-actions)
  + [Resources](#policy-auto-scaling-resources)
  + [Condition keys](#policy-auto-scaling-condition-keys)
+ [Amazon EC2 Auto Scaling resource\-based policies](#security_iam_service-with-iam-resource-based-policies)
+ [Access Control Lists \(ACLs\)](#security_iam_service-with-iam-acls)
+ [Authorization based on Amazon EC2 Auto Scaling tags](#security_iam_service-with-iam-tags)
+ [Amazon EC2 Auto Scaling IAM roles](#security_iam_service-with-iam-roles)
  + [Use temporary credentials with Amazon EC2 Auto Scaling](#security_iam_service-with-iam-roles-tempcreds)
  + [Service\-linked roles](#security_iam_service-with-iam-roles-service-linked)
  + [Service roles](#security_iam_service-with-iam-roles-service)
  + [Choose an IAM role in Amazon EC2 Auto Scaling](#security_iam_service-with-iam-roles-choose)
+ [Learn more about IAM permission policies](#auto-scaling-resources-about-iam)

## Amazon EC2 Auto Scaling identity\-based policies<a name="security_iam_service-with-iam-id-based-policies"></a>

With IAM identity\-based policies, you can specify allowed or denied actions and resources, and the conditions under which actions are allowed or denied\. Amazon EC2 Auto Scaling supports specific actions, resources, and condition keys\. To learn about all of the elements that you use in a JSON policy, see [IAM JSON policy elements reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) in the *IAM User Guide*\.

### Actions<a name="security_iam_service-with-iam-id-based-policies-actions"></a>

Administrators can use AWS JSON policies to specify who has access to what\. That is, which **principal** can perform **actions** on what **resources**, and under what **conditions**\.

The `Action` element of a JSON policy describes the actions that you can use to allow or deny access in a policy\. Policy actions usually have the same name as the associated AWS API operation\. There are some exceptions, such as *permission\-only actions* that don't have a matching API operation\. There are also some operations that require multiple actions in a policy\. These additional actions are called *dependent actions*\.

Include actions in a policy to grant permissions to perform the associated operation\.

Policy actions in Amazon EC2 Auto Scaling use the following prefix before the action: `autoscaling:`\. Policy statements must include either an `Action` or `NotAction` element\. Amazon EC2 Auto Scaling defines its own set of actions that describe tasks that you can perform with this service\.

To specify multiple actions in a single statement, separate them with commas as shown in the following example\.

```
"Action": [
      "autoscaling:CreateAutoScalingGroup",
      "autoscaling:UpdateAutoScalingGroup"
```

You can specify multiple actions using wildcards \(\*\)\. For example, to specify all actions that begin with the word `Describe`, include the following action\.

```
"Action": "autoscaling:Describe*"
```

To see a list of Amazon EC2 Auto Scaling actions, see [Actions](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_Operations.html) in the *Amazon EC2 Auto Scaling API Reference*\.

### Resources<a name="policy-auto-scaling-resources"></a>

Administrators can use AWS JSON policies to specify who has access to what\. That is, which **principal** can perform **actions** on what **resources**, and under what **conditions**\.

The `Resource` JSON policy element specifies the object or objects to which the action applies\. Statements must include either a `Resource` or a `NotResource` element\. As a best practice, specify a resource using its [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)\. You can do this for actions that support a specific resource type, known as *resource\-level permissions*\.

For actions that don't support resource\-level permissions, such as listing operations, use a wildcard \(\*\) to indicate that the statement applies to all resources\.

```
"Resource": "*"
```

You can restrict access to specific Auto Scaling groups and launch configurations by using their ARNs to identify the resource that the IAM policy applies to\. For more information about the format of ARNs, see [Amazon Resource Names \(ARNs\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) in the *AWS General Reference*\.

An Auto Scaling group has the following ARN\.

```
"Resource": "arn:aws:autoscaling:region:123456789012:autoScalingGroup:uuid:autoScalingGroupName/asg-name"
```

A launch configuration has the following ARN\.

```
"Resource": "arn:aws:autoscaling:region:123456789012:launchConfiguration:uuid:launchConfigurationName/lc-name"
```

To specify an Auto Scaling group with the `CreateAutoScalingGroup` action, you must replace the UUID with `*` as shown in the following\.

```
"Resource": "arn:aws:autoscaling:region:123456789012:autoScalingGroup:*:autoScalingGroupName/asg-name"
```

To specify a launch configuration with the `CreateLaunchConfiguration` action, you must replace the UUID with `*` as shown in the following\.

```
"Resource": "arn:aws:autoscaling:region:123456789012:launchConfiguration:*:launchConfigurationName/lc-name"
```

Not all Amazon EC2 Auto Scaling actions support resource\-level permissions\. For actions that don't support resource\-level permissions, you must use "`*`" as the resource\.

The following Amazon EC2 Auto Scaling actions do not support resource\-level permissions\.
+ `DescribeAccountLimits`
+ `DescribeAdjustmentTypes`
+ `DescribeAutoScalingGroups`
+ `DescribeAutoScalingInstances`
+ `DescribeAutoScalingNotificationTypes`
+ `DescribeInstanceRefreshes`
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
+ `DescribeWarmPool`

To learn with which actions you can specify the ARN of each resource, see [Actions, resources, and condition keys for Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonec2autoscaling.html) in the *IAM User Guide*\.

### Condition keys<a name="policy-auto-scaling-condition-keys"></a>

Administrators can use AWS JSON policies to specify who has access to what\. That is, which **principal** can perform **actions** on what **resources**, and under what **conditions**\.

The `Condition` element \(or `Condition` *block*\) lets you specify conditions in which a statement is in effect\. The `Condition` element is optional\. You can create conditional expressions that use [condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html), such as equals or less than, to match the condition in the policy with values in the request\. 

If you specify multiple `Condition` elements in a statement, or multiple keys in a single `Condition` element, AWS evaluates them using a logical `AND` operation\. If you specify multiple values for a single condition key, AWS evaluates the condition using a logical `OR` operation\. All of the conditions must be met before the statement's permissions are granted\.

 You can also use placeholder variables when you specify conditions\. For example, you can grant an IAM user permission to access a resource only if it is tagged with their IAM user name\. For more information, see [IAM policy elements: variables and tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html) in the *IAM User Guide*\. 

AWS supports global condition keys and service\-specific condition keys\. To see all AWS global condition keys, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

Amazon EC2 Auto Scaling defines its own set of condition keys and also supports using some global condition keys\. To see all AWS global condition keys, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

The following condition keys are specific to Amazon EC2 Auto Scaling:
+ `autoscaling:ImageId`
+ `autoscaling:InstanceType`
+ `autoscaling:InstanceTypes`
+ `autoscaling:LaunchConfigurationName`
+ `autoscaling:LaunchTemplateVersionSpecified`
+ `autoscaling:LoadBalancerNames`
+ `autoscaling:MaxSize`
+ `autoscaling:MetadataHttpEndpoint`
+ `autoscaling:MetadataHttpPutResponseHopLimit`
+ `autoscaling:MetadataHttpTokens`
+ `autoscaling:MinSize`
+ `autoscaling:ResourceTag/key`
+ `autoscaling:SpotPrice`
+ `autoscaling:TargetGroupARNs`
+ `autoscaling:VPCZoneIdentifiers`

To learn with which actions and resources you can use a condition key, see [Actions, resources, and condition keys for Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonec2autoscaling.html) in the *IAM User Guide*\.

## Amazon EC2 Auto Scaling resource\-based policies<a name="security_iam_service-with-iam-resource-based-policies"></a>

Other Amazon Web Services, such as Amazon Simple Storage Service, support resource\-based permissions policies\. For example, you can attach a permissions policy to an S3 bucket to manage access permissions to that bucket\.

Amazon EC2 Auto Scaling does not support resource\-based policies\.

## Access Control Lists \(ACLs\)<a name="security_iam_service-with-iam-acls"></a>

Amazon EC2 Auto Scaling does not support Access Control Lists \(ACLs\)\. 

## Authorization based on Amazon EC2 Auto Scaling tags<a name="security_iam_service-with-iam-tags"></a>

You can apply tag\-based, resource\-level permissions in the identity\-based policies that you create for Amazon EC2 Auto Scaling\. This gives you better control over which resources a user can create, modify, use, or delete\.

To use tags with IAM policies, you provide tag information in the [condition element](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) of a policy using the following condition keys:
+ Use `autoscaling:ResourceTag/tag-key: tag-value` to allow \(or deny\) user actions on Auto Scaling groups with specific tags\. 
+ Use `aws:RequestTag/tag-key: tag-value` to require that a specific tag be present \(or not present\) in a request\. 
+ Use `aws:TagKeys [tag-key, ...]` to require that specific tag keys be present \(or not present\) in a request\. 

To see examples of identity\-based policies based on tags, see [Amazon EC2 Auto Scaling identity\-based policy examples](security_iam_id-based-policy-examples.md)\. 

For more information, see [Tag Auto Scaling groups and instances](ec2-auto-scaling-tagging.md)\.

## Amazon EC2 Auto Scaling IAM roles<a name="security_iam_service-with-iam-roles"></a>

An [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is an entity within your AWS account that has specific permissions\.

### Use temporary credentials with Amazon EC2 Auto Scaling<a name="security_iam_service-with-iam-roles-tempcreds"></a>

You can use temporary credentials to sign in with federation, assume an IAM role, or to assume a cross\-account role\. You obtain temporary security credentials by calling AWS STS API operations such as [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) or [GetFederationToken](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetFederationToken.html)\. 

Amazon EC2 Auto Scaling supports using temporary credentials\. 

### Service\-linked roles<a name="security_iam_service-with-iam-roles-service-linked"></a>

Service\-linked roles allow Amazon Web Services to access resources in other services to complete an action on your behalf\. Service\-linked roles appear in your IAM account and are owned by the service\. An IAM administrator can view but not edit the permissions for service\-linked roles\.

Amazon EC2 Auto Scaling supports service\-linked roles\. For details about creating or managing Amazon EC2 Auto Scaling service\-linked roles, see [Service\-linked roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)\.

### Service roles<a name="security_iam_service-with-iam-roles-service"></a>

This feature allows a service to assume a [service role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-role) on your behalf\. This role allows the service to access resources in other services to complete an action on your behalf\. An IAM administrator can change the permissions for this role\. However, doing so might break the functionality of the service\.

Amazon EC2 Auto Scaling supports service roles for lifecycle hook notifications\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\.

### Choose an IAM role in Amazon EC2 Auto Scaling<a name="security_iam_service-with-iam-roles-choose"></a>

If you have previously created an IAM role that your applications running on Amazon EC2 can assume, you can choose this role when you create a launch template or launch configuration\. Amazon EC2 Auto Scaling provides you with a list of roles to choose from\. When creating these roles, it's important to associate least privilege IAM policies that restrict access to the specific API calls that the application requires\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\. 

## Learn more about IAM permission policies<a name="auto-scaling-resources-about-iam"></a>

Use the following topics to learn more about creating IAM permission policies to control who can or cannot use specific API actions\.
+ Amazon EC2 Auto Scaling
  + [Actions, resources, and condition keys for Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonec2autoscaling.html) in the *IAM User Guide*
  + [Amazon EC2 Auto Scaling identity\-based policy examples](security_iam_id-based-policy-examples.md)
+ Amazon EC2 launch templates
  + [Actions, resources, and condition keys for Amazon EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonec2.html) in the *IAM User Guide*
  + [Launch template support](ec2-auto-scaling-launch-template-permissions.md)