# Troubleshoot Amazon EC2 Auto Scaling: Launch templates<a name="ts-as-launch-template"></a>

Use the following information to help you diagnose and fix common permission issues that you might encounter when using a launch template with your Auto Scaling group\.

**Note**  
For other errors that you might receive during Auto Scaling group creation that are associated with your launch template, see [Troubleshoot Amazon EC2 Auto Scaling: EC2 instance launch failures](ts-as-instancelaunchfailure.md)\.

## You are not authorized to perform this operation<a name="ts-launch-template-unauthorized-error"></a>

**Problem**: When you try to specify a launch template, you get the "You are not authorized to perform this operation" error\. 

**Cause 1**: If you are attempting to use a launch template, and the IAM credentials that you are using do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. 

**Solution 1**: Verify that the IAM user or role that you are using to make the request has permissions to call the EC2 API actions you need, including the `ec2:RunInstances` action\. If you specified any tags in your launch template, you must also have permission to use the `ec2:CreateTags` action\. For a topic that shows how you might create your own policy to allow just what you want your IAM user or role to be able to do, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

**Solution 2**: Verify that your IAM user or role is using the `AmazonEC2FullAccess` policy\. This AWS managed policy grants full access to all Amazon EC2 resources and related services, including Amazon EC2 Auto Scaling, CloudWatch, and Elastic Load Balancing\. 

**Cause 2**: If you are attempting to use a launch template that specifies an instance profile, you must have IAM permission to pass the IAM role that is associated with the instance profile\. 

**Solution 3**: Verify that the IAM user or role that you are using to make the request has the correct permissions to pass the specified role to the Amazon EC2 Auto Scaling service\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\. 

For further troubleshooting topics related to instance profiles, see [Troubleshooting Amazon EC2 and IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_iam-ec2.html) in the *IAM User Guide*\.