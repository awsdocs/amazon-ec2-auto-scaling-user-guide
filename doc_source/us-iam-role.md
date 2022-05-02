# IAM role for applications that run on Amazon EC2 instances<a name="us-iam-role"></a>

Applications that run on Amazon EC2 instances need credentials to access other Amazon Web Services\. To provide these credentials in a secure way, use an IAM role\. The role supplies temporary permissions that the application can use when it accesses other AWS resources\. The role's permissions determine what the application is allowed to do\.

For instances in an Auto Scaling group, you must create a launch template or launch configuration and choose an instance profile to associate with the instances\. An instance profile is a container for an IAM role that allows Amazon EC2 to pass the IAM role to an instance when the instance is launched\. First, create an IAM role that has all of the permissions required to access the AWS resources\. Then, create the instance profile and assign the role to it\.

**Note**  
As a best practice, we strongly recommend that you create the role so that it has the minimum permissions to other AWS services that your application requires\. 

**Contents**
+ [Prerequisites](#us-iam-role-prereq)
+ [Create a launch template](#us-iam-role-create-lt)
+ [Create a launch configuration](#us-iam-role-create-launch)
+ [See also](#iam-role-see-also)

## Prerequisites<a name="us-iam-role-prereq"></a>

Create the IAM role that your application running on Amazon EC2 can assume\. Choose the appropriate permissions so that the application that is subsequently given the role can make the specific API calls that it needs\. 

If you use the IAM console instead of the AWS CLI or one of the AWS SDKs, the console creates an instance profile automatically and gives it the same name as the role to which it corresponds\. <a name="create-iam-role-console"></a>

**To create an IAM role \(console\)**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane on the left, choose **Roles**\.

1. Choose **Create role**\.

1. For **Select trusted entity**, choose **AWS service**\. 

1. For your use case, choose **EC2** and then choose **Next**\. 

1. If possible, select the policy to use for the permissions policy or choose **Create policy** to open a new browser tab and create a new policy from scratch\. For more information, see [Creating IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html#access_policies_create-start) in the *IAM User Guide*\. After you create the policy, close that tab and return to your original tab\. Select the check box next to the permissions policies that you want the service to have\.

1. \(Optional\) Set a permissions boundary\. This is an advanced feature that is available for service roles\. For more information, see [Permissions boundaries for IAM entities ](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) in the *IAM User Guide*\.

1. Choose **Next**\.

1. On the **Name, review, and create** page, for **Role name**, enter a role name to help you identify the purpose of this role\. This name must be unique within your AWS account\. Because other AWS resources might reference the role, you can't edit the name of the role after it has been created\. 

1. Review the role, and then choose **Create role**\. 

Use IAM user permissions to control access to your new IAM role\. The `iam:PassRole` permission is needed on the IAM user who creates or updates an Auto Scaling group using a launch template that specifies an instance profile, or who creates a launch configuration that specifies an instance profile\. For an example policy that grants users specific permissions, see [Control which IAM roles can be passed \(using PassRole\)](security_iam_id-based-policy-examples.md#policy-example-pass-IAM-role)\.

## Create a launch template<a name="us-iam-role-create-lt"></a>

When you create the launch template using the AWS Management Console, in the **Advanced details** section, select the role from **IAM instance profile**\. For more information, see [Configure advanced settings for your launch template](create-launch-template.md#advanced-settings-for-your-launch-template)\.

When you create the launch template using the [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command from the AWS CLI, specify the instance profile name of your IAM role as shown in the following example\.

```
aws ec2 create-launch-template --launch-template-name my-lt-with-instance-profile --version-description version1 \
--launch-template-data '{"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro","IamInstanceProfile":{"Name":"my-instance-profile"}}'
```

## Create a launch configuration<a name="us-iam-role-create-launch"></a>

When you create the launch configuration using the AWS Management Console, in the **Additional configuration** section, select the role from **IAM instance profile**\. For more information, see [Create a launch configuration](create-launch-config.md)\.

When you create the launch configuration using the [create\-launch\-configuration](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command from the AWS CLI, specify the instance profile name of your IAM role as shown in the following example\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc-with-instance-profile \
--image-id ami-04d5cc9b88example --instance-type t2.micro \
--iam-instance-profile my-instance-profile
```

## See also<a name="iam-role-see-also"></a>

For more information to help you start learning about and using IAM roles for Amazon EC2, see: 
+ [IAM roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) in the *Amazon EC2 User Guide for Linux Instances*
+ [Using instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) and [Using an IAM role to grant permissions to applications running on Amazon EC2 instances](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) in the *IAM User Guide*