# Troubleshoot Amazon EC2 Auto Scaling: Launch templates<a name="ts-as-launch-template"></a>

Use the following information to help you diagnose and fix common issues that you might encounter when trying to specify a launch template for your Auto Scaling group\.

**Can't launch instances**  
If you are unable to launch any instances with an already specified launch template, check the following for general troubleshooting: [Troubleshoot Amazon EC2 Auto Scaling: EC2 instance launch failures](ts-as-instancelaunchfailure.md)\.

## You must use a valid fully\-formed launch template \(invalid value\)<a name="ts-launch-template-invalid-error"></a>

**Problem**: When you try to specify a launch template for an Auto Scaling group, you get the `You must use a valid fully-formed launch template` error\. You might encounter this error because the values in the launch template are only validated when an Auto Scaling group that is using the launch template is created or updated\.

**Cause 1**: If you receive a `You must use a valid fully-formed launch template` error, then there are issues that cause Amazon EC2 Auto Scaling to consider something about the launch template not valid\. This is a generic error that can have several different causes\. 

**Solution 1**: Try the following steps to troubleshoot:

1. Pay attention to the second part of the error message to find more information\. Following the `You must use a valid fully-formed launch template` error, see the more specific error message that identifies the issue that you will need to address\.

1. If you are unable to find the cause, test your launch template with the [RunInstances](https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html) command\. Use the `--dry-run` option, as shown in the following example\. This lets you reproduce the issue and can provide insights about its cause\.

   ```
   aws ec2 run-instances --launch-template LaunchTemplateName=my-template,Version='1' --dry-run
   ```

1. If a value is not valid, verify that the specified resource exists and that it's correct\. For example, when you specify an Amazon EC2 key pair, the resource must exist in your account and in the Region in which you are creating or updating your Auto Scaling group\.

1. If expected information is missing, verify your settings and adjust the launch template as needed\.

1. After making your changes, re\-run the [RunInstances](https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html) command with the `--dry-run` option to verify that your launch template uses valid values\.

For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\.

## You are not authorized to use launch template \(insufficient IAM permissions\)<a name="ts-launch-template-unauthorized-error"></a>

**Problem**: When you try to specify a launch template for an Auto Scaling group, you get the `You are not authorized to use launch template` error\. 

**Cause 1**: If you are attempting to use a launch template, and the IAM credentials that you are using do not have sufficient permissions, you receive an error that you're not authorized to use the launch template\. 

**Solution 1**: Verify that the IAM credentials that you are using to make the request has permissions to call the EC2 API actions you need, including the `ec2:RunInstances` action\. If you specified any tags in your launch template, you must also have permission to use the `ec2:CreateTags` action\.

**Solution 2**: Verify that the IAM credentials that you are using to make the request is assigned the `AmazonEC2FullAccess` policy\. This AWS managed policy grants full access to all Amazon EC2 resources and related services, including Amazon EC2 Auto Scaling, CloudWatch, and Elastic Load Balancing\.

**Cause 2**: If you are attempting to use a launch template that specifies an instance profile, you must have IAM permission to pass the IAM role that is associated with the instance profile\.

**Solution 3**: Verify that the IAM credentials that you are using to make the request has the correct `iam:PassRole` permission to pass the specified role to the Amazon EC2 Auto Scaling service\. For more information and an example IAM policy, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md) and [Control which IAM roles can be passed \(using PassRole\)](security_iam_id-based-policy-examples.md#policy-example-pass-IAM-role)\. For further troubleshooting topics related to instance profiles, see [Troubleshooting Amazon EC2 and IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_iam-ec2.html) in the *IAM User Guide*\.

**Cause 3**: If you are attempting to use a launch template that specifies an AMI in another AWS account, and the AMI is private and not shared with the AWS account you are using, you receive an error that you're not authorized to use the launch template\.

**Solution 4**: Verify that the permissions on the AMI include the account that you are using\. For more information, see [Share an AMI with specific AWS accounts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sharingamis-explicit.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**Important**  
For more information about permissions for launch templates, including example IAM policies, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.