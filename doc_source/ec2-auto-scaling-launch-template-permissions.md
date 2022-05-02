# Launch template support<a name="ec2-auto-scaling-launch-template-permissions"></a>

Amazon EC2 Auto Scaling supports using Amazon EC2 launch templates with your Auto Scaling groups\. We recommend that you allow users to create Auto Scaling groups from launch templates, because doing so allows them to use the latest features of Amazon EC2 Auto Scaling and Amazon EC2\. In addition, users must specify a launch template to use a [mixed instances policy](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_MixedInstancesPolicy.html)\.

You can use the `AmazonEC2FullAccess` policy to give users complete access to work with Amazon EC2 Auto Scaling resources, launch templates, and other EC2 resources in their account\. Or, you can create your own custom IAM policies to give users fine\-grained permissions to work with launch templates, as described in this topic\. 

**A sample policy that you can tailor for your own use**

The following shows an example of a permissions policy that you can tailor for your own use\. The policy allows IAM users to create, modify, and delete all Auto Scaling groups, but only if the group uses the tag `environment=test`\. It then gives permission for all Describe actions\. Because Describe actions do not support resource\-level permissions, you must specify them in a separate statement without conditions\. 

Users with this policy have permission to create or update an Auto Scaling group using a launch template because they're also given permission to use the `ec2:RunInstances` action\. 

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
                "autoscaling:Describe*",
                "ec2:RunInstances"
            ],
            "Resource": "*"
        }
    ]
}
```

`ec2:RunInstances` are checked when an Auto Scaling group is created or updated using a launch template\. If you want to restrict access to the resources that are used to launch an instance or otherwise limit what IAM users can do, you must modify this policy to add your own statements that filter these permissions\. 

The following examples show policy statements that you could use to control the permissions that users have when using launch templates\. 

**Contents**
+ [Require launch templates that have a specific tag](#policy-example-launch-template-ex1)
+ [Require a launch template and a version number](#policy-example-launch-template-ex2)
+ [Require the use of instance metadata service version 2 \(IMDSv2\)](#instance-metadata-requireIMDSv2)
+ [Restrict access to Amazon EC2 resources](#policy-example-launch-template-ex4)
+ [Permissions required to tag instances and volumes](#policy-example-launch-template-createtags)
+ [Additional permissions](#policy-launch-template-additional-permissions)

## Require launch templates that have a specific tag<a name="policy-example-launch-template-ex1"></a>

The following example restricts access to calling the `ec2:RunInstances` action with launch templates that are located in the specified Region and that have the tag `environment=test`\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": "arn:aws:ec2:region:123456789012:launch-template/*",
            "Condition": {
                "StringEquals": { "ec2:ResourceTag/environment": "test" }
            }
        }
    ]
}
```

## Require a launch template and a version number<a name="policy-example-launch-template-ex2"></a>

The following example allows users to create and modify Auto Scaling groups if they specify the version number of the launch template, and then denies permission to create or modify any Auto Scaling groups using a launch configuration\. If users with this policy omit the version number to specify either the `Latest` or `Default` launch template version, or specify a launch configuration instead, the action fails\. 

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
                "Bool": { "autoscaling:LaunchTemplateVersionSpecified": "true" }
            }
        },
        {
            "Effect": "Deny",
            "Action": [
                "autoscaling:CreateAutoScalingGroup",
                "autoscaling:UpdateAutoScalingGroup"
            ],
            "Resource": "*",
            "Condition": {
                "Null": { "autoscaling:LaunchConfigurationName": "false" }
            }
        }
    ]
}
```

## Require the use of instance metadata service version 2 \(IMDSv2\)<a name="instance-metadata-requireIMDSv2"></a>

For extra security, you can set your users' permissions to require the use of a launch template that requires IMDSv2\. For more information, see [Configuring the instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

The following example specifies that users can't call the `ec2:RunInstances` action unless the instance is also opted in to require the use of IMDSv2 \(indicated by `"ec2:MetadataHttpTokens":"required"`\)\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RequireImdsV2",
            "Effect": "Deny",
            "Action": "ec2:RunInstances",
            "Resource": "arn:aws:ec2:*:*:instance/*",
            "Condition": {
                "StringNotEquals": { "ec2:MetadataHttpTokens": "required" }
            }
        }
    ]
}
```

**Tip**  
To force replacement Auto Scaling instances to launch that use a new launch template or a new version of a launch template with the instance metadata options configured, you can terminate existing instances in the group\. Amazon EC2 Auto Scaling immediately starts launching new instances to replace the instances that you terminated\. Alternatively, you can start an instance refresh to do a rolling update of your group\. For more information, see [Replace Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\. 

## Restrict access to Amazon EC2 resources<a name="policy-example-launch-template-ex4"></a>

The following example controls the configuration of the instances that a user can launch by restricting access to Amazon EC2 resources\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": [
                "arn:aws:ec2:region:123456789012:subnet/subnet-1a2b3c4d",
                "arn:aws:ec2:region:123456789012:security-group/sg-903004f88example",
                "arn:aws:ec2:region:123456789012:network-interface/*",
                "arn:aws:ec2:region:123456789012:volume/*",
                "arn:aws:ec2:region::image/ami-04d5cc9b88example"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": "arn:aws:ec2:region:123456789012:instance/*",
            "Condition": {
                "StringEquals": { "ec2:InstanceType": "t2.micro" }
            }
        }
    ]
}
```

In this example, there are two statements:
+ The first statement requires that users launch instances into a specific subnet \(`subnet-1a2b3c4d`\), using a specific security group \(`sg-903004f88example`\), and using a specific AMI \(`ami-04d5cc9b88example`\)\. It also gives users access to additional resources that they need to launch instances: network interfaces and volumes\. 
+ The second statement allows users to launch instances only of a specific instance type \(`t2.micro`\)\. 

## Permissions required to tag instances and volumes<a name="policy-example-launch-template-createtags"></a>

The following example allows users to tag instances and volumes on creation\. This policy is needed if there are tags specified in the launch template\. For more information, see [Grant permission to tag resources during creation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/supported-iam-actions-tagging.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:CreateTags",
            "Resource": "arn:aws:ec2:region:123456789012:*/*",
            "Condition": {
                "StringEquals": { "ec2:CreateAction": "RunInstances" }
            }
        }
    ]
}
```

## Additional permissions<a name="policy-launch-template-additional-permissions"></a>

Depending on which scenarios you want to support, you can specify these additional actions in the `Action` element of an IAM policy statement\. 

For a user to have the ability to pass an IAM role to provisioned instances, that user must have permission for `iam:PassRole`\. You can use an `iam:PassRole` policy to allow \(or deny\) users to pass a role to Amazon EC2 if the launch template specifies an instance profile with an IAM role\. For an example policy, see [Control which IAM roles can be passed \(using PassRole\)](security_iam_id-based-policy-examples.md#policy-example-pass-IAM-role)\.

You must give your console users permissions for the `ec2:DescribeLaunchTemplates` and `ec2:DescribeLaunchTemplateVersions` actions\. Without these permissions, launch template data cannot load in the Auto Scaling group wizard, and users cannot step through the wizard to launch instances using a launch template\. 

To control access to the `ec2:CreateLaunchTemplate` and `ec2:CreateLaunchTemplateVersion` actions, see [Control the use of launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#launch-template-permissions) and [Example: Work with launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html#iam-example-launch-templates) in the *Amazon EC2 User Guide for Linux Instances*\. 

**Note**  
For groups that are configured to use the `Latest` or `Default` launch template version, permissions for actions to be completed when launching instances are not checked by Amazon EC2 Auto Scaling when a new version of the launch template is created\. This is an important consideration when setting up your permissions for who can create and manage launch template versions\. 