# Service\-Linked Roles for Amazon EC2 Auto Scaling<a name="autoscaling-service-linked-role"></a>

Amazon EC2 Auto Scaling uses service\-linked roles for the permissions that it requires to call other AWS services on your behalf\. For more information, see [Using Service\-Linked Roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the *IAM User Guide*\.

## Permissions Granted by AWSServiceRoleForAutoScaling<a name="service-linked-role-permissions"></a>

By default, Amazon EC2 Auto Scaling uses the **AWSServiceRoleForAutoScaling** service\-linked role to make the following calls on your behalf\. Alternatively, you can create a service\-linked role for Amazon EC2 Auto Scaling to use to make the following calls on your behalf\.

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

If you specify encrypted EBS volumes for your Auto Scaling instances and you use customer managed CMKs for encryption, you must grant the appropriate service\-linked role access to the CMKs so that the Auto Scaling group can launch instances on your behalf\. The principal is the Amazon Resource Name \(ARN\) of the service\-linked role\. When launching On\-Demand instances, use a service\-linked role for Amazon EC2 Auto Scaling\. When launching Spot Instances, use the **AWSServiceRoleForEC2Spot** role when using a launch configuration and a service\-linked role for Amazon EC2 Auto Scaling when using a launch template\. For more information, see [Using Key Policies in AWS KMS](http://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)\.

You must configure permissions to allow an IAM entity \(such as a user, group, or role\) to create, edit, or delete a service\-linked role\. For more information, see [Service\-Linked Role Permissions](http://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#service-linked-role-permissions) in the *IAM User Guide*\.

## Create the Service\-Linked Role<a name="create-service-linked-role"></a>

Amazon EC2 Auto Scaling creates the **AWSServiceRoleForAutoScaling** service\-linked role for you the first time that you create an Auto Scaling group but do not specify a service\-linked role\.

If you created an Auto Scaling group before March 2018, when Amazon EC2 Auto Scaling began supporting service\-linked roles, Amazon EC2 Auto Scaling created the **AWSServiceRoleForAutoScaling** role in your AWS account\. For more information, see [A New Role Appeared in My AWS Account](http://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_roles.html#troubleshoot_roles_new-role-appeared) in the *IAM User Guide*\.

Alternatively, you can specify a service\-linked role that you've created when you create your Auto Scaling group\. This enables you to grant different service\-linked roles access to different KMS keys\. You can also track which Auto Scaling group made an API call in your CloudTrail logs by noting the service\-linked role in use\.

Use the following [create\-service\-linked\-role](http://docs.aws.amazon.com/cli/latest/reference/iam/create-service-linked-role.html) command to create a service\-linked role for Amazon EC2 Auto Scaling with the name **AWSServiceRoleForAutoScaling**\_*suffix*\. The suffix helps you identify the purpose of the role\.

```
aws iam create-service-linked-role --aws-service-name autoscaling.amazonaws.com --custom-suffix suffix
```

The output of this command includes the ARN of the service\-linked role, which you can use to grant the service\-linked role access to your KMS keys\.

To create the service\-linked role using the AWS Management Console, see [Creating a Service\-Linked Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#create-service-linked-role) in the *IAM User Guide*\.

## Edit the Service\-Linked Role<a name="edit-service-linked-role"></a>

With the **AWSServiceRoleForAutoScaling** role created by Amazon EC2 Auto Scaling, you can edit its description using IAM\. You can edit any service\-linked role that you created for use with your Auto Scaling groups\. For more information, see [Editing a Service\-Linked Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

## Delete the Service\-Linked Role<a name="delete-service-linked-role"></a>

If you no longer need to use an Auto Scaling group, we recommend that you delete its service\-linked role\.

You can delete a service\-linked role only after first deleting the related AWS resources\. If a service\-linked role is used with multiple Auto Scaling groups, you must delete all Auto Scaling groups that use the service\-linked role before you can delete it\. This protects your resources because you can't inadvertently remove permission to access them\. For more information, see [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)\.

You can use IAM to delete the default service\-linked role or one that you've created\. For more information, see [Deleting a Service\-Linked Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#delete-service-linked-role) in the *IAM User Guide*\.

After you delete the **AWSServiceRoleForAutoScaling** service\-linked role, Amazon EC2 Auto Scaling will create the role again when you create an Auto Scaling group but do not specify a different service\-linked role\.