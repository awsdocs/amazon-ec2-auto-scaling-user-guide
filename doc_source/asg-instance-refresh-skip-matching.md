# Use an instance refresh with skip matching<a name="asg-instance-refresh-skip-matching"></a>

Skip matching tells Amazon EC2 Auto Scaling to ignore instances that already have your latest updates\. This way, you don't replace more instances than you need to\. This is helpful when you want to make sure that your Auto Scaling group uses a particular version of your launch template and only replaces those instances that use a different version\.

The following considerations apply to skip matching:
+ If you start an instance refresh with both skip matching and a *desired configuration*, Amazon EC2 Auto Scaling checks to see if any instances match your desired configuration\. Then, it only replaces the instances that don't match your desired configuration\. After the instance refresh succeeds, Amazon EC2 Auto Scaling updates the group to reflect your desired configuration\. 
+ If you start an instance refresh with skip matching, but you don't specify a desired configuration, Amazon EC2 Auto Scaling checks to see if any instances match the configuration that you last saved on the Auto Scaling group\. Then, it only replaces the instances that don't match your last saved configuration\.
+ You can use skip matching with a new launch template, a new version of a launch template, or a set of instance types\. If you enable skip matching, but none of these are changing, the instance refresh will succeed immediately without replacing any instances\. If you made any other changes in your desired configuration \(such as changing your Spot allocation strategy\), Amazon EC2 Auto Scaling waits for the instance refresh to succeed\. Then, it updates the Auto Scaling group settings to reflect the new desired configuration\. 
+ You cannot use skip matching with a new launch configuration\.
+ During an instance refresh with skip matching, if either `$Default` or `$Latest` is specified for the launch template, Amazon EC2 Auto Scaling re\-evaluates the launch template version as each instance is replaced\. Therefore, if you create a new version of your launch template while the instances are still being replaced, Amazon EC2 Auto Scaling can end up using the new version of your launch template\. This can cause previously replaced instances to be replaced again as Amazon EC2 Auto Scaling applies the new version across the group\. 

This section includes AWS CLI instructions for starting an instance refresh with skip matching enabled\. For instructions on using the console, see [Start an instance refresh \(console\)](start-instance-refresh.md#start-instance-refresh-console)\.

## Skip matching \(basic procedure\)<a name="skip-matching"></a>

Follow the steps in this section to use the AWS CLI to do the following:
+ Create the launch template that you want to apply to your instances\.
+ Start an instance refresh to apply your launch template to your Auto Scaling group\. If you do not enable skip matching, all instances will be replaced\. This is true even if the launch template used to provision the instance is the same as the one that you specified for your desired configuration\.

**To use skip matching with a new launch template**

1. Use the [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command to create a new launch template for your Auto Scaling group\. Include the `--launch-template-data` option and JSON input that defines the details of the instances that are created for your Auto Scaling group\.

   For example, use the following command to create a basic launch template with the AMI ID *`ami-0123456789abcdef0`* and the `t2.micro` instance type\.

   ```
   aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
     --launch-template-data '{"ImageId":"ami-0123456789abcdef0","InstanceType":"t2.micro"}'
   ```

   If successful, the command returns output similar to the following\.

   ```
   {
      "LaunchTemplate": {
        "LaunchTemplateId": "lt-068f72b729example",
        "LaunchTemplateName": "my-template-for-auto-scaling",
        "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
        "CreateTime": "2023-01-30T18:16:06.000Z",
        "DefaultVersionNumber": 1,
        "LatestVersionNumber": 1
     }
   }
   ```

   For more information, see [Examples for creating and managing launch templates with the AWS Command Line Interface \(AWS CLI\)](examples-launch-templates-aws-cli.md)\.

1. Use the [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command to initiate the instance replacement workflow and apply your new launch template with the ID *`lt-068f72b729example`*\. Because the launch template is new, it only has one version\. This means that version `1` of the launch template is the target of this instance refresh\. If a scale\-out event occurs during the instance refresh, and Amazon EC2 Auto Scaling provisions new instances using the version `1` of this launch template, they will not be replaced\. On successful completion of the operation, the new launch template is successfully applied to your Auto Scaling group\. 

   ```
   aws autoscaling start-instance-refresh --cli-input-json file://config.json
   ```

   Contents of `config.json`\.

   ```
   {
       "AutoScalingGroupName": "my-asg",
       "DesiredConfiguration": {
         "LaunchTemplate": {
             "LaunchTemplateId": "lt-068f72b729example",
             "Version": "$Default"
          }
       },
       "Preferences": {
         "SkipMatching": true
       }
   }
   ```

   If successful, the command returns output similar to the following\.

   ```
   {
     "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
   }
   ```

## Skip matching \(mixed instances group\)<a name="skip-matching-mixed-instances-group"></a>

If you have an Auto Scaling group with a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md), follow the steps in this section to use the AWS CLI to start an instance refresh with skip matching\. You have the following options:
+ Provide a new launch template to apply to all instance types specified in the policy\.
+ Provide an updated set of instance types with or without changing the launch template in the policy\. For example, you might want to migrate away from unwanted instance types\. You would use the launch template as is, without changing the AMI, security groups, or other specifics of the instances being replaced\.

Follow the steps in one of the following sections, depending on which option fits your needs\.

**To use skip matching with a new launch template**

1. Use the [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command to create a new launch template for your Auto Scaling group\. Include the `--launch-template-data` option and JSON input that defines the details of the instances that are created for your Auto Scaling group\.

   For example, use the following command to create a launch template with the AMI ID *`ami-0123456789abcdef0`*\.

   ```
   aws ec2 create-launch-template --launch-template-name my-new-template --version-description version1 \
     --launch-template-data '{"ImageId":"ami-0123456789abcdef0"}'
   ```

   If successful, the command returns output similar to the following\.

   ```
   {
      "LaunchTemplate": {
        "LaunchTemplateId": "lt-04d5cc9b88example",
        "LaunchTemplateName": "my-new-template",
        "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
        "CreateTime": "2023-01-31T15:56:02.000Z",
        "DefaultVersionNumber": 1,
        "LatestVersionNumber": 1
     }
   }
   ```

   For more information, see [Examples for creating and managing launch templates with the AWS Command Line Interface \(AWS CLI\)](examples-launch-templates-aws-cli.md)\.

1. To view the existing mixed instances policy for your Auto Scaling group, run the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\. You'll need this information in the next step, when you start the instance refresh\.

   The following example command returns the mixed instances policy configured for the Auto Scaling group named *`my-asg`*\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

   If successful, the command returns output similar to the following\.

   ```
   {
     "AutoScalingGroups":[
       {
         "AutoScalingGroupName":"my-asg",
         "AutoScalingGroupARN":"arn",
         "MixedInstancesPolicy":{
           "LaunchTemplate":{
             "LaunchTemplateSpecification":{
               "LaunchTemplateId":"lt-073693ed27example",
               "LaunchTemplateName":"my-old-template",
               "Version":"$Default"
             },
             "Overrides":[
               {
                 "InstanceType":"c5.large"
               },
               {
                 "InstanceType":"c5a.large"
               },
               {
                 "InstanceType":"m5.large"
               },
               {
                 "InstanceType":"m5a.large"
               }
             ]
           },
           "InstancesDistribution":{
             "OnDemandAllocationStrategy":"prioritized",
             "OnDemandBaseCapacity":1,
             "OnDemandPercentageAboveBaseCapacity":50,
             "SpotAllocationStrategy":"price-capacity-optimized"
           }
         },
         "MinSize":1,
         "MaxSize":5,
         "DesiredCapacity":4,
         ...
       }
     ]
   }
   ```

1. Use the [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command to initiate the instance replacement workflow and apply your new launch template with the ID *`lt-04d5cc9b88example`*\. Because the launch template is new, it only has one version\. This means that version `1` of the launch template is the target of this instance refresh\. If a scale\-out event occurs during the instance refresh, and Amazon EC2 Auto Scaling provisions new instances using the version `1` of this launch template, they will not be replaced\. On successful completion of the operation, the updated mixed instances policy is successfully applied to your Auto Scaling group\.

   ```
   aws autoscaling start-instance-refresh --cli-input-json file://config.json
   ```

   Contents of `config.json`\.

   ```
   {
     "AutoScalingGroupName":"my-asg",
     "DesiredConfiguration":{
       "MixedInstancesPolicy":{
         "LaunchTemplate":{
           "LaunchTemplateSpecification":{
             "LaunchTemplateId":"lt-04d5cc9b88example",
             "Version":"$Default"
           },
           "Overrides":[
             {
               "InstanceType":"c5.large"
             },
             {
               "InstanceType":"c5a.large"
             },
             {
               "InstanceType":"m5.large"
             },
             {
               "InstanceType":"m5a.large"
             }
           ]
         },
         "InstancesDistribution":{
           "OnDemandAllocationStrategy":"prioritized",
           "OnDemandBaseCapacity":1,
           "OnDemandPercentageAboveBaseCapacity":50,
           "SpotAllocationStrategy":"price-capacity-optimized"
           }
         }
       }
     },
     "Preferences":{
       "SkipMatching":true
     }
   }
   ```

   If successful, the command returns output similar to the following\.

   ```
   {
     "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
   }
   ```

In this next procedure, you provide an updated set of instance types without changing the launch template\. 

**To use skip matching with an updated set of instance types**

1. To view the existing mixed instances policy for your Auto Scaling group, run the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command\. You'll need this information in the next step, when you start the instance refresh\.

   The following example command returns the mixed instances policy configured for the Auto Scaling group named *`my-asg`*\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

   If successful, the command returns output similar to the following\.

   ```
   {
     "AutoScalingGroups":[
       {
         "AutoScalingGroupName":"my-asg",
         "AutoScalingGroupARN":"arn",
         "MixedInstancesPolicy":{
           "LaunchTemplate":{
             "LaunchTemplateSpecification":{
               "LaunchTemplateId":"lt-073693ed27example",
               "LaunchTemplateName":"my-template-for-auto-scaling",
               "Version":"$Default"
             },
             "Overrides":[
               {
                 "InstanceType":"c5.large"
               },
               {
                 "InstanceType":"c5a.large"
               },
               {
                 "InstanceType":"m5.large"
               },
               {
                 "InstanceType":"m5a.large"
               }
             ]
           },
           "InstancesDistribution":{
             "OnDemandAllocationStrategy":"prioritized",
             "OnDemandBaseCapacity":1,
             "OnDemandPercentageAboveBaseCapacity":50,
             "SpotAllocationStrategy":"price-capacity-optimized"
           }
         },
         "MinSize":1,
         "MaxSize":5,
         "DesiredCapacity":4,
         ...
       }
     ]
   }
   ```

1. Use the [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command to initiate the instance replacement workflow and apply your updates\. If you want to replace instances that use specific instance types, your desired configuration must specify the mixed instance policy with only the instance types that you want\. You can choose whether to add new instance types in their place\.

   The following example command starts an instance refresh without the unwanted instance type *`m5a.large`*\. When an instance type in your group doesn’t match one of the remaining three instance types, the instances are replaced\. \(Note that an instance refresh does not choose the instance types from which to provision the new instances; instead, the [allocation strategies](ec2-auto-scaling-mixed-instances-groups.md#allocation-strategies) do that\.\) On successful completion of the operation, the updated mixed instances policy is successfully applied to your Auto Scaling group\.

   ```
   aws autoscaling start-instance-refresh --cli-input-json file://config.json
   ```

   Contents of `config.json` 

   ```
   {
     "AutoScalingGroupName":"my-asg",
     "DesiredConfiguration":{
       "MixedInstancesPolicy":{
         "LaunchTemplate":{
           "LaunchTemplateSpecification":{
             "LaunchTemplateId":"lt-073693ed27example",
             "Version":"$Default"
           },
           "Overrides":[
             {
               "InstanceType":"c5.large"
             },
             {
               "InstanceType":"c5a.large"
             },
             {
               "InstanceType":"m5.large"
             }
           ]
         },
         "InstancesDistribution":{
           "OnDemandAllocationStrategy":"prioritized",
           "OnDemandBaseCapacity":1,
           "OnDemandPercentageAboveBaseCapacity":50,
           "SpotAllocationStrategy":"price-capacity-optimized"
           }
         }
       }
     },
     "Preferences":{
       "SkipMatching":true
     }
   }
   ```