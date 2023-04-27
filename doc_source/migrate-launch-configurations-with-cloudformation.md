# Migrate AWS CloudFormation stacks from launch configurations to launch templates<a name="migrate-launch-configurations-with-cloudformation"></a>

You can migrate your existing AWS CloudFormation stack templates from launch configurations to launch templates\. To do this, add a launch template directly to an existing stack template and then associate the launch template with the Auto Scaling group in the stack template\. Then, use your modified template to update your stack\. 

When migrating to launch templates, this topic saves you time by providing instructions for rewriting the launch configurations in your CloudFormation stack templates as launch templates\. 

**Topics**
+ [Find Auto Scaling groups that use a launch configuration](#find-auto-scaling-groups-to-migrate)
+ [Update a stack to use a launch template](#update-stack-to-use-launch-template)
+ [Understand update behavior of stack resources](#understand-update-behavior)
+ [Track the migration](#track-the-migration)
+ [Launch configuration mapping reference](#launch-configuration-mapping-reference)

## Find Auto Scaling groups that use a launch configuration<a name="find-auto-scaling-groups-to-migrate"></a>

**To find Auto Scaling groups that use a launch configuration**
+ Use the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to list the names of Auto Scaling groups that are using launch configurations in the specified Region\. Include the `--filters` option to narrow the results to groups associated with a CloudFormation stack \(by filtering by the `aws:cloudformation:stack-name` tag key\)\. 

  ```
  aws autoscaling describe-auto-scaling-groups --region REGION \
    --filters Name=tag-key,Values=aws:cloudformation:stack-name \
    --query 'AutoScalingGroups[?LaunchConfigurationName!=`null`].AutoScalingGroupName'
  ```

  The following shows example output\.

  ```
  [
      "{stack-name}-group-1",
      "{stack-name}-group-2",
      "{stack-name}-group-3"
  ]
  ```

  You can find additional useful AWS CLI commands for finding Auto Scaling groups to migrate and filtering the output in [Migrate to launch templates](launch-templates.md#migrate-to-launch-templates)\.

**Important**  
If your stack resources have `AWSEB` in their name, this means they were created through AWS Elastic Beanstalk\. In this case, you must update the Beanstalk environment to direct Elastic Beanstalk to remove the launch configuration and replace it with a launch template\.

## Update a stack to use a launch template<a name="update-stack-to-use-launch-template"></a>

Follow the steps in this section to do the following:
+ Rewrite the launch configuration as a launch template using the equivalent launch template properties\. 
+ Associate the new launch template with the Auto Scaling group\. 
+ Deploy these updates\.

**To modify the stack template and update the stack**

1. Follow the same general procedures for modifying the stack template described in [Modifying a stack template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html) in the *AWS CloudFormation User Guide*\.

1. Rewrite the launch configuration as a launch template\. See the following example: 

   **Example: A simple launch configuration**

   ```
   ---
   Resources:
     myLaunchConfig: 
       Type: AWS::AutoScaling::LaunchConfiguration
       Properties: 
         ImageId: ami-02354e95b3example
         InstanceType: t3.micro
         SecurityGroups:
           - !Ref EC2SecurityGroup
         KeyName: MyKeyPair
         BlockDeviceMappings: 
           - DeviceName: /dev/xvda
             Ebs: 
               VolumeSize: 150
               DeleteOnTermination: true
         IamInstanceProfile: !Ref RootInstanceProfile
         UserData: 
           Fn::Base64: !Sub |
             #!/bin/bash -xe
             yum install -y aws-cfn-bootstrap
             /opt/aws/bin/cfn-signal -e $? --stack !Ref 'AWS::StackName' --resource myASG --region !Ref 'AWS::Region'
   ```

   **Example: The launch template equivalent**

   ```
   ---
   Resources:
     myLaunchTemplate:
       Type: AWS::EC2::LaunchTemplate
       Properties:
         LaunchTemplateName: !Sub ${AWS::StackName}-launch-template
         LaunchTemplateData:
           ImageId: ami-02354e95b3example
           InstanceType: t3.micro
           SecurityGroupIds:
             - Ref! EC2SecurityGroup
           KeyName: MyKeyPair
           BlockDeviceMappings: 
             - DeviceName: /dev/xvda
               Ebs: 
                 VolumeSize: 150
                 DeleteOnTermination: true
           IamInstanceProfile: !Ref RootInstanceProfile
           UserData: 
             Fn::Base64: !Sub |
               #!/bin/bash -x
               yum install -y aws-cfn-bootstrap
               /opt/aws/bin/cfn-signal -e $? --stack !Ref 'AWS::StackName' --resource myASG --region !Ref 'AWS::Region'
   ```

   For reference information for all the properties that Amazon EC2 supports, see [AWS::EC2::LaunchTemplate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-launchtemplate.html) in the *AWS CloudFormation User Guide*\. 

   Note how the launch template includes the `LaunchTemplateName` property with a value of `!Sub ${AWS::StackName}-launch-template`\. This is required if you want the name of the launch template to include the stack name\.

1. If the `AssociatePublicIpAddress`, `InstanceMonitoring`, or `PlacementTenancy` properties are present in your launch configuration, you must convert these to a structure\. For examples, see [AWS::EC2::LaunchTemplate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-launchtemplate.html)\. 

   An exception is when the value for the `MapPublicIpOnLaunch` property on the subnets you used for your Auto Scaling group matches the value for the `AssociatePublicIpAddress` property in your launch configuration\. In this case, you can ignore the `AssociatePublicIpAddress` property\. The `AssociatePublicIpAddress` property is only used to override the `MapPublicIpOnLaunch` property to change whether instances receive a public IPv4 address at launch\. 

1. You can copy security groups from the `SecurityGroups` property to one of two places in your launch template\. Normally, you copy the security groups to the `SecurityGroupIds` property\. However, if you create a `NetworkInterfaces` structure within your launch template to specify the `AssociatePublicIpAddress` property, then you must copy the security groups to the `Groups` property of the network interface instead\.

1. If any `BlockDeviceMapping` structures are present in your launch configuration with `NoDevice` set to `true`, then you must specify an empty string for `NoDevice` in your launch template to have Amazon EC2 omit the device\.

1. If the `SpotPrice` property is present in your launch configuration, we recommend that you omit it from your launch template\. Your Spot Instances will launch at the current Spot price\. This price will never exceed the On\-Demand price\.

   To request Spot Instances, you have two mutually exclusive options:
   + The first is to use the `InstanceMarketOptions` structure in your launch template \(not recommended\)\. For more information, see [AWS::EC2::LaunchTemplate InstanceMarketOptions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-launchtemplate-launchtemplatedata-instancemarketoptions.html) in the *AWS CloudFormation User Guide*\.
   + The other is to add a `MixedInstancesPolicy` structure to your Auto Scaling group\. Doing so provides you with more options for how you make the request\. A Spot Instance request in your launch template doesn't support more than one instance type selection per Auto Scaling group\. However, a mixed instances policy does support more than one instance type selection per Auto Scaling group\. Spot Instance requests benefit from having more than one instance type to choose from\. For more information, see [AWS::AutoScaling::AutoScalingGroup MixedInstancesPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-autoscaling-autoscalinggroup-mixedinstancespolicy.html) in the *AWS CloudFormation User Guide*\.

1. Remove the `LaunchConfigurationName` property from the [AWS::AutoScaling::AutoScalingGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-group.html) resource\. Add the launch template in its place\.

   In the following examples, the [Ref](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html) intrinsic function gets the ID of the [AWS::EC2::LaunchTemplate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-launchtemplate.html) resource with the logical ID `myLaunchTemplate`\. The [GetAtt](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html) function gets the latest version number \(for example, `1`\) of the launch template for the `Version` property\.

   **Example: Without a mixed instances policy**

   ```
   ---
   Resources:
     myASG:
       Type: AWS::AutoScaling::AutoScalingGroup
       Properties:
         LaunchTemplate:
           LaunchTemplateId: !Ref myLaunchTemplate
           Version: !GetAtt myLaunchTemplate.LatestVersionNumber
         ...
   ```

   **Example: With a mixed instances policy**

   ```
   ---
   Resources:
     myASG:
       Type: AWS::AutoScaling::AutoScalingGroup
       Properties:        
         MixedInstancesPolicy:
           LaunchTemplate:
             LaunchTemplateSpecification:
               LaunchTemplateId: !Ref myLaunchTemplate
               Version: !GetAtt myLaunchTemplate.LatestVersionNumber
         ...
   ```

   For reference information for all the properties that Amazon EC2 Auto Scaling supports, see [AWS::AutoScaling::AutoScalingGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-group.html) in the *AWS CloudFormation User Guide*\.

1. When you are ready to deploy these updates, follow the CloudFormation procedures to update the stack with your modified stack template\. For more information, see [Modifying a stack template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html) in the *AWS CloudFormation User Guide*\. 

## Understand update behavior of stack resources<a name="understand-update-behavior"></a>

CloudFormation updates stack resources by comparing changes between the updated template you provide, and resource configurations you described in the previous version of your stack template\. Resource configurations that haven't changed remain unaffected during the update process\.

CloudFormation supports the [UpdatePolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html) attribute for Auto Scaling groups\. During an update, if `UpdatePolicy` is set to `AutoScalingRollingUpdate`, CloudFormation replaces `InService` instances after you perform the steps in this procedure\. If `UpdatePolicy` is set to `AutoScalingReplacingUpdate`, CloudFormation replaces the Auto Scaling group and its warm pool \(if one exists\)\.

If you didn't specify an `UpdatePolicy` attribute for your Auto Scaling group, the launch template is checked for correctness, but CloudFormation does not deploy any change across the instances in the Auto Scaling group\. All new instances will use your launch template, but existing instances continue to run with the launch configuration that they were originally launched with \(despite the launch configuration not existing\)\. The exception is when you change your purchase options, for example, by adding a mixed instances policy\. In this case, your Auto Scaling group gradually replaces the existing instances with new instances to match the new purchase options\.

## Track the migration<a name="track-the-migration"></a>

**To track the migration**

1. In the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation), select the stack that you updated and then choose the **Events** tab to view the stacks events\.

1. To update the event list with the most recent events, choose the refresh button in the CloudFormation console\.

1. While your stack is updating, you will notice multiple events for each resource update\. If you see an exception in the **Status reason** column that indicates a problem when trying to create the launch template, see [Troubleshoot Amazon EC2 Auto Scaling: Launch templates](ts-as-launch-template.md) for potential causes\.

1. \(Optional\) Depending on your use of the `UpdatePolicy` attribute, you can monitor the progress of your Auto Scaling group from the [Auto Scaling groups page](https://console.aws.amazon.com/ec2/v2/home?#AutoScalingGroups) of the Amazon EC2 console\. Select the Auto Scaling group\. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched or terminated instances, or whether the scaling activity is still in progress\. 

1. When the stack update is complete, CloudFormation issues an `UPDATE_COMPLETE` stack event\. For more information, see [Monitoring the progress of a stack update](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-monitor-stack.html) in the *AWS CloudFormation User Guide*\. 

1. After the stack update is complete, open the [Launch templates page](https://console.aws.amazon.com/ec2/v2/#LaunchTemplates) and [Launch configurations page](https://console.aws.amazon.com/ec2/v2/home?#LaunchConfigurations) of the Amazon EC2 console\. You will notice a new launch template is created, and the launch configuration is deleted\. 

## Launch configuration mapping reference<a name="launch-configuration-mapping-reference"></a>

For reference purposes, the following table lists all the top\-level properties in the [AWS::AutoScaling::LaunchConfiguration](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-autoscaling-launchconfiguration.html) resource with their corresponding property in the [AWS::EC2::LaunchTemplate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-launchtemplate.html) resource\.


| Launch configuration source property | Launch template target property | 
| --- | --- | 
| AssociatePublicIpAddress | NetworkInterfaces\.AssociatePublicIpAddress | 
| BlockDeviceMappings | BlockDeviceMappings | 
| ClassicLinkVPCId | Not available¹ | 
| ClassicLinkVPCSecurityGroups | Not available¹ | 
| EbsOptimized | EbsOptimized | 
| IamInstanceProfile | IamInstanceProfile | 
| ImageId | ImageId | 
| InstanceId | InstanceId | 
| InstanceMonitoring | Monitoring\.Enabled | 
| InstanceType | InstanceType | 
| KernelId | KernelId | 
| KeyName | KeyName | 
| LaunchConfigurationName | LaunchTemplateName | 
| MetadataOptions | MetadataOptions | 
| PlacementTenancy | Placement\.Tenancy  | 
| RamDiskId | RamDiskId | 
| SecurityGroups | Either SecurityGroupIds or NetworkInterfaces\.Groups, but not both | 
| SpotPrice | InstanceMarketOptions\.SpotOptions\.MaxPrice | 
| UserData | UserData | 

¹ The `ClassicLinkVPCId` and `ClassicLinkVPCSecurityGroups` properties are not available to use in a launch template\. If your launch configuration has these properties, you can use this opportunity to migrate from EC2\-Classic to VPC\. For more information, see [Migrate from EC2\-Classic to a VPC](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/vpc-migrate.html) in the *Amazon EC2 User Guide for Linux Instances*\.