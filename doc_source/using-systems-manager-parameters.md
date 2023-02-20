# Using AWS Systems Manager parameters instead of AMI IDs in launch templates<a name="using-systems-manager-parameters"></a>

Amazon EC2 Auto Scaling supports using AWS Systems Manager parameters that reference Amazon Machine Image \(AMI\) IDs in launch templates\. With Systems Manager parameters, you can update your Auto Scaling groups to use new AMI IDs without needing to create new launch templates or new versions of launch templates each time an AMI ID changes\. These IDs can change regularly, such as when an AMI is updated with the latest operating system or software updates\.

You can create, update, or delete your Systems Manager parameters using the Parameter Store, a capability of AWS Systems Manager\. You must create a Systems Manager parameter before you can use it in a launch template\. To get started, create a parameter with the data type `aws:ec2:image`, and for its value, enter the ID of an AMI\. The AMI ID has the form `ami-<identifier>`, for example, `ami-123example456`\. The correct AMI ID depends on the instance type and AWS Region that you're launching your Auto Scaling group in\. 

**Important**  
To specify Systems Manager parameters in launch templates, you must have permission to use the `ssm:GetParameters` action\. You also need permission to use the `ssm:GetParameters` action to use a launch template that specifies a Systems Manager parameter\. This allows the parameter value to be validated\. For example IAM policies, see [Restricting access to Systems Manager parameters using IAM policies](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-access.html) in the *AWS Systems Manager User Guide*\.

For more information, see the following resources:
+ To create a parameter in the console, see [Create a Systems Manager parameter \(console\)](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-create-console.html) in the *AWS Systems Manager User Guide*\. 
+ To create a parameter with the AWS CLI, see [Create a Systems Manager parameter \(AWS CLI\)](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html) in the *AWS Systems Manager User Guide*\.
+ To create parameter versions and labels, see [Working with parameter versions](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-versions.html) and [Working with parameter labels](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-labels.html) in the *AWS Systems Manager User Guide*\. 
+ For information about making sure that the parameter value you enter is a valid AMI ID, see [Native parameter support for Amazon Machine Image IDs](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-ec2-aliases.html) in the *AWS Systems Manager User Guide*\. 
+ For more information about using parameters that reference AMI IDs in launch templates, see [Use a Systems Manager parameter instead of an AMI ID](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-launch-template.html#use-an-ssm-parameter-instead-of-an-ami-id) in the *Amazon EC2 User Guide for Linux Instances*\.
+ You can also reference AWS Systems Manager public parameters for public AMIs maintained by AWS in your launch templates\. To learn how to reference these public parameters, see [Find the latest Amazon Linux AMI using Systems Manager](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html#finding-an-ami-parameter-store) in the *Amazon EC2 User Guide for Linux Instances* and [Find the latest Amazon Linux AMI using Systems Manager](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/finding-an-ami.html#finding-an-ami-parameter-store) in the *Amazon EC2 User Guide for Windows Instances*\.

------
#### [ Console ]

**To create a launch template using an AWS Systems Manager parameter**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Launch Templates**, and then choose **Create launch template**\. 

1. For **Launch template name**, enter a descriptive name for the launch template\.

1. Under **Application and OS Images \(Amazon Machine Image\)**, choose **Browse more AMIs**\.

1. Choose the arrow button to the right of the search bar, and then choose **Specify custom value/Systems Manager parameter**\.

1. In the **Specify custom value or Systems Manager parameter** dialog box, do the following:

   1. For **AMI ID or Systems Manager parameter string**, enter the Systems Manager parameter name using one of the following formats:
      + **resolve:ssm:*parameter\-name***
      + **resolve:ssm:*parameter\-name*:*version\-number***
      + **resolve:ssm:*parameter\-name*:*label***
      + **resolve:ssm:*public\-parameter***

   1. Choose **Save**\.

1. Specify any other launch template parameters as needed, and then choose **Create launch template**\.

For more information about creating a launch template from the console, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\. 

------
#### [ AWS CLI ]

**Example launch template that specifies a customer\-owned parameter**

1. Use the following syntax: `resolve:ssm:parameter-name`, where `resolve:ssm` is the standard prefix and `parameter-name` is the Systems Manager parameter name\.

   The following example creates a launch template that gets the AMI ID from an existing Systems Manager parameter named `golden-ami`\. 

   ```
   aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling \
     --launch-template-data file://config.json
   ```

   Contents of `config.json`:

   ```
   { 
       "ImageId":"resolve:ssm:golden-ami",
       "InstanceType":"t2.micro"
   }
   ```

   The default version of the parameter, if it is not specified, is the latest version\. 

   The following example references a specific version of the `golden-ami` parameter\. The example uses version `3` of the `golden-ami` parameter, but you can use any valid version number\. 

   ```
   { 
       "ImageId":"resolve:ssm:golden-ami:3",
       "InstanceType":"t2.micro"
   }
   ```

   The following similar example references the parameter label `prod` that maps to a specific version of the `golden-ami` parameter\.

   ```
   { 
       "ImageId":"resolve:ssm:golden-ami:prod",
       "InstanceType":"t2.micro"
   }
   ```

   The following is example output\. 

   ```
   {
       "LaunchTemplate": {
           "LaunchTemplateId": "lt-068f72b724example",
           "LaunchTemplateName": "my-template-for-auto-scaling",
           "CreateTime": "2022-12-27T17:11:21.000Z",
           "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
           "DefaultVersionNumber": 1,
           "LatestVersionNumber": 1
       }
   }
   ```

1. To confirm that the launch template gets your preferred AMI ID, use the [describe\-launch\-template\-versions](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html) command\. The command uses the `--resolve-alias` option to resolve the parameter to the actual AMI ID\.

   ```
   aws ec2 describe-launch-template-versions --launch-template-name my-template-for-auto-scaling \
      --versions $Default --resolve-alias
   ```

   The example returns the AMI ID for `ImageId`\. When an instance is launched using this launch template, the AMI ID resolves to `ami-04d5cc9b88example`\.

   ```
   {
       "LaunchTemplateVersions": [
           {
               "LaunchTemplateId": "lt-068f72b724example",
               "LaunchTemplateName": "my-template-for-auto-scaling",
               "VersionNumber": 1,
               "CreateTime": "2022-12-27T17:11:21.000Z",
               "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
               "DefaultVersion": true,
               "LaunchTemplateData": {
                   "ImageId": "ami-04d5cc9b88example",
                   "InstanceType": "t2.micro"
               }
           }
       ]
   }
   ```

**Example launch template that specifies an AWS\-owned public parameter**

1. Use the following syntax: `resolve:ssm:public-parameter`, where `resolve:ssm` is the standard prefix and `public-parameter` is the path and name of the public parameter\.

   In this example, the launch template uses an AWS\-provided public parameter to launch instances using the latest Amazon Linux 2 AMI in the AWS Region that is configured for your profile\. 

   ```
   aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
     --launch-template-data file://config.json
   ```

   Contents of `config.json`:

   ```
   { 
       "ImageId":"resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2",
       "InstanceType":"t2.micro"
   }
   ```

   The following is an example response\.

   ```
   {
       "LaunchTemplate": {
           "LaunchTemplateId": "lt-089c023a30example",
           "LaunchTemplateName": "my-template-for-auto-scaling",
           "CreateTime": "2022-12-28T19:52:27.000Z",
           "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
           "DefaultVersionNumber": 1,
           "LatestVersionNumber": 1
       }
   }
   ```

1. To confirm that the launch template gets the correct AMI ID, use the [describe\-launch\-template\-versions](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html) command\. The command uses the `--resolve-alias` option to resolve the parameter to the actual AMI ID\.

   ```
   aws ec2 describe-launch-template-versions --launch-template-name my-template-for-auto-scaling \
      --versions $Default --resolve-alias
   ```

   The example returns the AMI ID for `ImageId`\. When an instance is launched using this launch template, the AMI ID resolves to `ami-0ac394d6a3example`\.

   ```
   {
       "LaunchTemplateVersions": [
           {
               "LaunchTemplateId": "lt-089c023a30example",
               "LaunchTemplateName": "my-template-for-auto-scaling",
               "VersionNumber": 1,
               "CreateTime": "2022-12-28T19:52:27.000Z",
               "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
               "DefaultVersion": true,
               "LaunchTemplateData": {
                   "ImageId": "ami-0ac394d6a3example",
                   "InstanceType": "t2.micro",
               }
           }
       ]
   }
   ```

------

## Limitations<a name="using-systems-manager-parameters-limitations"></a>
+ Currently, you cannot use a launch template that specifies a Systems Manager parameter instead of an AMI ID on an Auto Scaling group with a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md)\.
+ The Systems Manager Parameter Store parameters that you create must exist in the same account that the instances are run in\.
+ Amazon EC2 Auto Scaling only supports specifying AMI IDs as parameters\.
+ While your Auto Scaling group has a launch template that references a parameter instead of an AMI ID, you cannot start an instance refresh that specifies a desired configuration or uses skip matching to replace the instances in the group\.
+ On each call to create or update your Auto Scaling group, Amazon EC2 Auto Scaling will resolve the Systems Manager parameter in the launch template\. If you are using advanced parameters or higher throughput limits, the frequent calls to the Parameter Store \(that is, the `GetParameters` operation\) can increase your costs for Systems Manager because charges are incurred per Parameter Store API interaction\. For more information, see [AWS Systems Manager pricing](http://aws.amazon.com/systems-manager/pricing/)\. 