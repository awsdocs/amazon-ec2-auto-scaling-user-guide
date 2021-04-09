# Troubleshooting Amazon EC2 Auto Scaling: Launch templates<a name="ts-as-launch-template"></a>

Use the following information to help you diagnose and fix common issues that you might encounter when using a launch template with your Auto Scaling group\.

## You are not authorized to perform this operation<a name="ts-launch-template-unauthorized-error"></a>

**Problem**: When you try to specify a launch template, you get the "You are not authorized to perform this operation" error\. 

**Cause 1**: If you are attempting to use a launch template, and the IAM credentials that you are using do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. 

**Solution 1**: Verify that the IAM user or role that you are using to make the request has the correct permissions\. For information about the permissions necessary to work with launch templates, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

**Cause 2**: If you are attempting to use a launch template that specifies an instance profile, you must have permission to pass the IAM role that is associated with the instance profile\. 

**Solution 2**: Verify that the IAM user or role that you are using to make the request has the correct permissions to pass the specified role to the Amazon EC2 Auto Scaling service\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\. For further troubleshooting topics related to instance profiles, see [Troubleshooting Amazon EC2 and IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_iam-ec2.html) in the *IAM User Guide*\.