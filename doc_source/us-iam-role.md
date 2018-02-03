# Launch Auto Scaling Instances with an IAM Role<a name="us-iam-role"></a>

AWS Identity and Access Management \(IAM\) roles for EC2 instances make it easier for you to access other AWS services securely from within the EC2 instances\. EC2 instances launched with an IAM role automatically have AWS security credentials available\. 

You can launch your Auto Scaling instances with an IAM role to automatically enable applications running on your instances to securely access other AWS resources\. You do this by creating a launch configuration with an EC2 instance profile\. An instance profile is a container for an IAM role\. First, create an IAM role that has all the permissions required to access the AWS resources, then add your role to the instance profile\.

For more information about IAM roles and instance profiles, see [IAM Roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) in the *IAM User Guide*\.

## Prerequisites<a name="us-iam-role-prereq"></a>

Create an IAM role for your EC2 instances\. The console creates an instance profile with the same name as the IAM role\.

**To create an IAM role**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**, **Create new role**\.

1. On the **Select role type** page, choose **Select** next to **Amazon EC2**\.

1. On the **Attach Policy** page, select an AWS managed policy that grants your instances access to the resources that they need\.

1. On the **Set role name and review** page, type a name for the role and choose **Create role**\.

## Create a Launch Configuration<a name="us-iam-role-create-launch"></a>

When you create the launch configuration using the AWS Management Console, on the **Configure Details** page, select the role from **IAM role**\. For more information, see [Creating a Launch Configuration](create-launch-config.md)\.

When you create the launch configuration using the [create\-launch\-configuration](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command from the AWS CLI, specify the name of the instance profile as follows:

```
aws autoscaling create-launch-configuration --launch-configuration-name my-lc-with-instance-profile \
--image-id ami-baba68d3 --instance-type m1.small \
--iam-instance-profile my-instance-profile
```