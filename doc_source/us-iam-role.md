# IAM role for applications that run on Amazon EC2 instances<a name="us-iam-role"></a>

Applications that run on Amazon EC2 instances need credentials to access other AWS services\. To provide these credentials in a secure way, use an IAM role\. The role supplies temporary permissions that the application can use when it accesses other AWS resources\. The role's permissions determine what the application is allowed to do\.

Applications running on the instances can access temporary credentials for the role through the instance profile metadata\. For more information, see [Using an IAM role to grant permissions to applications running on Amazon EC2 instances](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) in the *IAM User Guide*\.

For instances in an Auto Scaling group, you must create a launch template or launch configuration and choose an instance profile to associate with the instances\. An instance profile is a container for an IAM role that allows Amazon EC2 to pass the IAM role to an instance when the instance is launched\. First, create an IAM role that has all of the permissions required to access the AWS resources\. Then, create the instance profile and assign the role to it\. For more information, see [Using instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) in the *IAM User Guide*\.

**Note**  
When you use the IAM console to create a role for Amazon EC2, the console guides you through the steps for creating the role and automatically creates an instance profile with the same name as the IAM role\. 

For more information, see [IAM roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

## Prerequisites<a name="us-iam-role-prereq"></a>

Create the IAM role that your application running on Amazon EC2 can assume\. Choose the appropriate permissions so that the application that is subsequently given the role can make the specific API calls that it needs\. 

**Important**  
As a best practice, we strongly recommend that you create the role so that it has the minimum permissions to other AWS services that your application requires\. <a name="create-iam-role-console"></a>

**To create an IAM role**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**, **Create role**\.

1. For **Select type of trusted entity**, choose **AWS service**\. 

1. For **Choose the service that will use this role**, choose **EC2** and the **EC2** use case\. Choose **Next: Permissions**\. 

1. For **Attach permissions policies**, choose the AWS managed policies that contain the required permissions\. Choose **Next: Tags** and then **Next: Review**\.

1. On the **Review** page, enter a name for the role and choose **Create role**\. 

The `iam:PassRole` permission is needed on the IAM user who creates or updates an Auto Scaling group using a launch template that specifies an instance profile, or who creates a launch configuration that specifies an instance profile\. For an example policy, see [Control which IAM roles can be passed \(using PassRole\)](security_iam_id-based-policy-examples.md#policy-example-pass-IAM-role)\.

## Create a launch configuration<a name="us-iam-role-create-launch"></a>

When you create the launch configuration using the AWS Management Console, on the **Configure Details** page, select the role from **IAM role**\. For more information, see [Creating a launch configuration](create-launch-config.md)\.

When you create the launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command from the AWS CLI, specify the name of the instance profile as shown in the following example\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc-with-instance-profile \
--image-id ami-01e24be29428c15b2 --instance-type t2.micro \
--iam-instance-profile my-instance-profile
```

## Create a launch template<a name="us-iam-role-create-lt"></a>

When you create the launch template using the AWS Management Console, in the **Advanced Details** section, select the role from **IAM instance profile**\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.

When you create the launch template using the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command from the AWS CLI, specify the name of the instance profile as shown in the following example\.

```
aws ec2 create-launch-template --launch-template-name my-lt-with-instance-profile --version-description version1 \
--launch-template-data '{"ImageId":"ami-01e24be29428c15b2","InstanceType":"t2.micro","IamInstanceProfile":{"Name":"my-instance-profile"}}'
```