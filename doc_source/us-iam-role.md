# IAM Role for Applications that Run on Amazon EC2 Instances<a name="us-iam-role"></a>

Applications that run on Amazon EC2 instances need credentials in order to access other AWS services\. To provide credentials to the application in a secure way, use an IAM role\. 

When you launch an Amazon EC2 instance, you can specify an IAM role for the instance as a launch parameter\. Applications that run on the EC2 instance can use the role's credentials when they access AWS resources\. The role's permissions determine what the application is allowed to do\. 

For instances in an Auto Scaling group, you create a launch configuration or launch template with an EC2 *instance profile*\. An instance profile is a container for an IAM role\. First, create an IAM role that has all the permissions required to access the AWS resources, then add your role to the instance profile\.

For more information about IAM roles and instance profiles, see [Using an IAM Role to Grant Permissions to Applications Running on Amazon EC2 Instances ](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) in the *IAM User Guide*\.

## Prerequisites<a name="us-iam-role-prereq"></a>

Create an IAM role for your EC2 instances\. The console creates an instance profile with the same name as the IAM role\.<a name="create-iam-role-console"></a>

**To create an IAM role**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**, **Create new role**\.

1. On the **Select role type** page, choose **Select** next to **Amazon EC2**\.

1. On the **Attach Policy** page, select an AWS managed policy that grants your instances access to the resources that they need\.

1. On the **Set role name and review** page, type a name for the role and choose **Create role**\.

## Create a Launch Configuration<a name="us-iam-role-create-launch"></a>

When you create the launch configuration using the AWS Management Console, on the **Configure Details** page, select the role from **IAM role**\. For more information, see [Creating a Launch Configuration](create-launch-config.md)\.

When you create the launch configuration using the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command from the AWS CLI, specify the name of the instance profile as follows:

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc-with-instance-profile \
--image-id ami-baba68d3 --instance-type m1.small \
--iam-instance-profile my-instance-profile
```