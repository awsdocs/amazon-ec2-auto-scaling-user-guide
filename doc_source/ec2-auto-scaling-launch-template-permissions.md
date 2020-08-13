# Launch template support<a name="ec2-auto-scaling-launch-template-permissions"></a>

Amazon EC2 Auto Scaling supports using Amazon EC2 launch templates with your Auto Scaling groups\. We recommend that you allow users to create Auto Scaling groups from launch templates, because doing so allows them to use the latest features of EC2\. In addition, users must specify a launch template to use a [mixed instances policy](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_MixedInstancesPolicy.html)\.

You can use the `AmazonEC2FullAccess` policy to give users complete access to work with Amazon EC2 Auto Scaling resources, launch templates, and other EC2 resources in their AWS account\. Or, you can create your own custom IAM policies to give users fine\-grained permissions to work with specific API actions\. 

Use the following example policies to give users permissions to work with launch templates, unless another policy already grants them permission to do so\. You don't need to give these permissions to users who are only creating Auto Scaling groups from launch configurations \(an older feature of Amazon EC2 Auto Scaling\)\. 

For more information about IAM policies for Amazon EC2, see [IAM policies](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-policies-for-amazon-ec2.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**Topics**
+ [Example: Control access using resource\-level permissions](#policy-example-launch-template)
+ [Example: Allow users to create and manage launch templates and launch template versions](#policy-example-create-launch-template)
+ [Example: Require the use of instance metadata service version 2 \(IMDSv2\)](#instance-metadata-requireIMDSv2)
+ [Additional required permissions](#policy-example-console-user)

An IAM user or role that creates or updates an Auto Scaling group using a launch template must have permission to use the `ec2:RunInstances` action\. Users without this permission receive an error that they are not authorized to use the launch template\. 

Keep in mind that user permissions for `ec2:RunInstances` are only checked when an Auto Scaling group is created or updated using a launch template\. For an Auto Scaling group that is configured to use the `Latest` or `Default` launch template, the permissions are not checked when a new version of the launch template is created\. For permissions to be checked, users must configure the Auto Scaling group to use a specific version of the launch template\.

## Example: Control access using resource\-level permissions<a name="policy-example-launch-template"></a>

In granting user permissions, Amazon EC2 allows you to specify resource\-level permissions for resources that are created as part of a call to the `ec2:RunInstances` action, to control which resources users can work with\. This is a recommended practice\. 

The following example uses resource\-level permissions to restrict access to specific launch templates\. It also demonstrates some of the many possible ways that you can control the configuration of an instance that a user can launch when allowing the `ec2:RunInstances` permission\.

In this example, there are four statements:
+ The first statement restricts user access to launch templates that are located in the specified Region and that have the tag `environment=test`\. 
+ The second statement allows users to tag instances and volumes on creation\. This part is needed if there are tags specified in the launch template\.
+ The third statement requires that users launch instances into a specific subnet \(`subnet-1a2b3c4d`\), using a specific security group \(`sg-1a2b3c4d`\), and using a specific AMI \(`ami-1a2b3c4d`\)\. It also gives users access to additional resources that they need to launch instances: network interfaces and volumes\. 
+ The fourth statement allows users to launch instances only of a specific instance type \(`t2.micro`\)\. 

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
        },
        {
            "Effect": "Allow",
            "Action": "ec2:CreateTags",
            "Resource": "arn:aws:ec2:region:123456789012:*/*",
            "Condition": {
                "StringEquals": { "ec2:CreateAction": "RunInstances" }
            }
        },
        {
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": [
                "arn:aws:ec2:region:123456789012:subnet/subnet-1a2b3c4d",
                "arn:aws:ec2:region:123456789012:security-group/sg-1a2b3c4d",
                "arn:aws:ec2:region:123456789012:network-interface/*",
                "arn:aws:ec2:region:123456789012:volume/*",
                "arn:aws:ec2:region::image/ami-1a2b3c4d"
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

In addition, you can create an IAM policy to limit users' access so that they cannot use launch configurations when creating and updating Auto Scaling groups\. Optionally, you can also require that they set the specific version to use, to ensure that the IAM permissions for actions to be completed when launching instances are checked whenever Auto Scaling groups are updated to use a new version\. 

The following statements give users permissions to create or update an Auto Scaling group if they specify a launch template\. If they omit the version number to specify either the `Latest` or `Default` launch template version, or specify a launch configuration instead, the action fails\.

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

Alternatively, you can require that users specify either the `Latest` or `Default` version of the launch template, rather than a specific version of the template\. To do this, change the value of `autoscaling:LaunchTemplateVersionSpecified` to `false`, as shown in the following example\.

```
            "Condition": {
                "Bool": { "autoscaling:LaunchTemplateVersionSpecified": "false" }
            }
```

## Example: Allow users to create and manage launch templates and launch template versions<a name="policy-example-create-launch-template"></a>

You can create a policy that grants users permissions to create, modify, describe, and delete launch templates and launch template versions\. Before you add these permissions to a user, review the [Controlling the use of launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#launch-template-permissions) section of the *Amazon EC2 User Guide for Linux Instances*\. For additional example policies, see [Example: Working with launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html#iam-example-launch-templates) in the *Amazon EC2 User Guide for Linux Instances*\.

**Important**  
For groups that are configured to use the `Latest` or `Default` launch template version, permissions for actions to be completed when launching instances are not checked by Amazon EC2 Auto Scaling when a new version of the launch template is created\. This is an important consideration when setting up your permissions for who can create and manage launch template versions\.

The following policy gives users permissions to create launch templates\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:CreateLaunchTemplate",
            "Resource": "*"
        }
    ]
}
```

## Example: Require the use of instance metadata service version 2 \(IMDSv2\)<a name="instance-metadata-requireIMDSv2"></a>

**Important**  
If you need to require the use of IMDSv2 on all new instances, your Auto Scaling groups must use launch templates\. 

For extra security, you can set your users' permissions to require the use of a launch template that requires IMDSv2\. For more information, see [Configuring the instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

The following policy specifies that users can't call the `ec2:RunInstances` action unless the instance is also opted in to require the use of IMDSv2 \(indicated by `"ec2:MetadataHttpTokens":"required"`\)\. If they do not specify that the instance requires IMDSv2, they get an `UnauthorizedOperation` error when they call the `ec2:RunInstances` action\. 

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

To update an existing Auto Scaling group, make sure that it uses a new launch template with the instance metadata options configured, or uses a new version of the current launch template with the instance metadata options configured\. 

**Tip**  
To force replacement instances to launch that use your new launch template, you can terminate existing instances in the group\. Amazon EC2 Auto Scaling immediately starts launching new instances to replace the instances that you terminated\.   
Alternatively, if you use scaling policies, you can increase the desired capacity of the group to launch new instances\. If the policy conditions for scale in are met, the Auto Scaling group gradually terminates older instances \(depending on the termination policy of the group\)\. 

## Additional required permissions<a name="policy-example-console-user"></a>

Depending on which scenarios you want to support, you can specify these additional actions in the `Action` element of an IAM policy statement\.

To create and update Auto Scaling groups from the console, users must also have the following permissions from Amazon EC2:
+ `ec2:DescribeLaunchTemplates`
+ `ec2:DescribeLaunchTemplateVersions`
+ `ec2:DescribeVpcs`
+ `ec2:DescribeSubnets`
+ `ec2:DescribeAvailabilityZones`

Without these additional minimum permissions, launch template and network options cannot load in the Auto Scaling group wizard, and users cannot step through the wizard to launch instances using a launch template\. 

You can add more actions to your policy to give users more options in the wizard\. For example, you can add permissions for Elastic Load Balancing API actions to allow users to select from a list of existing load balancers in Step 3 of the wizard\.

Use `iam:PassRole` to allow \(or deny\) users to pass a role to Amazon EC2 if the launch template specifies an instance profile with an IAM role\. Users without this permission receive an error that they are not authorized to use the launch template\. For an example policy, see [Control which IAM roles can be passed \(using PassRole\)](security_iam_id-based-policy-examples.md#policy-example-pass-IAM-role)\.

To verify provisioning from the Amazon EC2 console, users might need additional permissions \(for example, `ec2:DescribeInstances` to view instances, `ec2:DescribeInstanceStatus` to show instance status, or `ec2:DescribeTags` to show tags\)\. 