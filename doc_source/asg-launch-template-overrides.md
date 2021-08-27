# Specifying a different launch template for an instance type<a name="asg-launch-template-overrides"></a>

The `Overrides` structure allows you to define a new launch template for individual instance types for a new or existing Auto Scaling group\. For example, if the architecture of an instance type requires a different AMI from the rest of the group, you must specify a launch template with a compatible AMI\. 

Say that you configure an Auto Scaling group for compute\-intensive applications and want to include a mix of C5, C5a, and C6g instance types\. However, C6g instances feature an AWS Graviton processor based on 64\-bit Arm architecture, while the C5 and C5a instances run on 64\-bit Intel x86 processors\. The AMI for C5 instances works on C5a instances and vice\-versa, but not on C6g instances\. The `Overrides` property allows you to include a different launch template for C6g instances, while still using the same launch template for C5 and C5a instances\.

**Note**  
Currently, this feature is available only if you use the AWS CLI or an SDK, and is not available from the console\.

## Adding or changing a launch template for an instance type \(AWS CLI\)<a name="launch-template-overrides-cli"></a>

The following procedure shows you how to use the AWS CLI to configure an Auto Scaling group so that one or more of the instance types uses a launch template that is different from the rest of the group\. 

**To create and configure a new Auto Scaling group**

1. Create a configuration file where you specify a mixed instances policy structure and include the `Overrides` structure\. 

   The following are the contents of an example configuration file formatted in JSON\. It specifies the `c5.large`, `c5a.large`, and `c6g.large` instance types and defines a new launch template for the `c6g.large` instance type to ensure that an appropriate AMI is used to launch Arm instances\. Amazon EC2 Auto Scaling uses the order of instance types to determine which instance type to use first when fulfilling On\-Demand capacity\.

   ```
   {
     "AutoScalingGroupName":"my-asg",
     "MixedInstancesPolicy":{
       "LaunchTemplate":{
         "LaunchTemplateSpecification":{
           "LaunchTemplateName":"my-launch-template-for-x86",
           "Version":"$Latest"
         },
         "Overrides":[
           {
             "InstanceType":"c6g.large",
             "LaunchTemplateSpecification": {
               "LaunchTemplateName": "my-launch-template-for-arm",
               "Version": "$Latest"
             }
           },
           {
             "InstanceType":"c5.large"
           },
           {
             "InstanceType":"c5a.large"
           }
         ]
       },
       "InstancesDistribution":{
         "OnDemandBaseCapacity": 1,
         "OnDemandPercentageAboveBaseCapacity": 50,
         "SpotAllocationStrategy": "capacity-optimized"
       }
     },
     "MinSize":1,
     "MaxSize":5,
     "DesiredCapacity":3,
     "VPCZoneIdentifier":"subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
     "Tags":[ ]
   }
   ```

1. Use the following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command, referencing the JSON file as the sole parameter for your Auto Scaling group\.

   ```
   aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
   ```

**To change the launch template for an instance type in an existing Auto Scaling group**
+ Use the following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to specify a different launch template for an instance type by passing the `Overrides` structure\. 

  When this change is made, any new instances that are launched are based on the new settings, but existing instances are not affected\. To ensure that your Auto Scaling group is using the new settings, you can replace all instances in the group by starting an instance refresh or by using the maximum instance lifetime feature\.

  ```
  aws autoscaling update-auto-scaling-group --cli-input-json file://~/config.json
  ```

  The following is an example `config.json` file\. 

  ```
  {
    "AutoScalingGroupName":"my-asg",
    "MixedInstancesPolicy":{
      "LaunchTemplate":{
        "Overrides":[
          {
            "InstanceType":"c6g.large",
            "LaunchTemplateSpecification": {
              "LaunchTemplateName": "my-launch-template-for-arm",
              "Version": "$Latest"
            }
          },
          {
            "InstanceType":"c5.large"
          },
          {
            "InstanceType":"c5a.large"
          }
        ]
      }
    }
  }
  ```

**To verify the launch templates for an Auto Scaling group**
+ Use the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify and view the currently specified launch templates\. 

  ```
  aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
  ```

  The following is an example response\.

  ```
  {
    "AutoScalingGroups":[
      {
        "AutoScalingGroupName":"my-asg",
        "AutoScalingGroupARN":"arn",
        "MixedInstancesPolicy":{
          "LaunchTemplate":{
            "LaunchTemplateSpecification":{
              "LaunchTemplateId":"lt-0fb0e487336917fb2",
              "LaunchTemplateName":"my-launch-template-for-x86",
              "Version":"$Latest"
            },
            "Overrides":[
              {
                "InstanceType":"c6g.large",
                "LaunchTemplateSpecification": {
                  "LaunchTemplateId": "lt-09d958b8fb2ba5bcc",
                    "LaunchTemplateName": "my-launch-template-for-arm",
                    "Version": "$Latest"
                }
              },
              {
                "InstanceType":"c5.large"
              },
              {
                "InstanceType":"c5a.large"
              }
            ]
          },
          "InstancesDistribution":{
            "OnDemandAllocationStrategy":"prioritized",
            "OnDemandBaseCapacity":1,
            "OnDemandPercentageAboveBaseCapacity":50,
            "SpotAllocationStrategy":"capacity-optimized"
          }
        },
        "MinSize":1,
        "MaxSize":5,
        "DesiredCapacity":3,
        "Instances":[
          {
            "InstanceId":"i-07c63168522c0f620",
            "InstanceType":"c5.large",
            "AvailabilityZone":"us-west-2c",
            "LifecycleState":"InService",
            "HealthStatus":"Healthy",
            "LaunchTemplate":{
              "LaunchTemplateId":"lt-0fb0e487336917fb2",
              "LaunchTemplateName":"my-launch-template-for-x86",
              "Version":"1"
            },
            "ProtectedFromScaleIn":false
          },
          {
            "InstanceId":"i-0b7ff78be9896a2c2",
            "InstanceType":"c5.large",
            "AvailabilityZone":"us-west-2a",
            "LifecycleState":"InService",
            "HealthStatus":"Healthy",
            "LaunchTemplate":{
              "LaunchTemplateId":"lt-0fb0e487336917fb2",
              "LaunchTemplateName":"my-launch-template-for-x86",
              "Version":"1"
            },
            "ProtectedFromScaleIn":false
          },
          {
            "InstanceId":"i-0c682c2ceae918bc0",
            "InstanceType":"c6g.large",
            "AvailabilityZone":"us-west-2b",
            "LifecycleState":"InService",
            "HealthStatus":"Healthy",
            "LaunchTemplate":{
              "LaunchTemplateId":"lt-09d958b8fb2ba5bcc",
              "LaunchTemplateName":"my-launch-template-for-arm",
              "Version":"1"
            },
        ...
      }
    ]
  }
  ```