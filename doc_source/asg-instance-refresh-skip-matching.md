# Instance refresh examples that enable skip matching with the AWS Command Line Interface \(AWS CLI\)<a name="asg-instance-refresh-skip-matching"></a>

By default, Amazon EC2 Auto Scaling can replace any instance in an Auto Scaling group during an instance refresh\. By enabling skip matching, you can avoid replacing instances that already have your desired configuration\. 

Skip matching makes it more efficient to:
+ Migrate from a launch configuration to the default or latest version of a launch template after launching one or more test instances\.
+ Migrate from unwanted instance types to instance types that are better suited for your application\.
+ Roll back changes after one or more instances are replaced as part of a failed or cancelled instance refresh\.

**Note**  
The skip matching feature cannot be used to update an Auto Scaling group that uses a launch configuration unless a launch template is specified for the desired configuration\.

The following AWS CLI examples demonstrate a few scenarios for the use of skip matching\. 

**Contents**
+ [Migrate to the default version of your launch template](#skip-matching-launch-template-latest)
+ [Migrate to the latest version of your launch template](#skip-matching-launch-template-default)
+ [Skip matching and mixed instances groups](#skip-matching-mixed-instances-group)
  + [Migrate to the default version of your launch template](#skip-matching-mixed-instances-group-launch-template-default)
  + [Migrate to the latest version of your launch template](#skip-matching-mixed-instances-group-launch-template-latest)
  + [Migrate away from unwanted instance types](#skip-matching-latest-generation-instance-type)

## Migrate to the default version of your launch template<a name="skip-matching-launch-template-latest"></a>

The following example shows a [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command that updates an Auto Scaling group to the default version of your launch template\. If there are any instances that are already using the default version of the specified launch template, they are not replaced\.

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

If successful, this command returns a JSON response that contains the instance refresh ID\. 

## Migrate to the latest version of your launch template<a name="skip-matching-launch-template-default"></a>

The following example shows a [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command that updates an Auto Scaling group to the latest version of your launch template\. If there are any instances that are already using the latest version of the specified launch template, they are not replaced\.

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
          "Version": "$Latest"
       }
    },
    "Preferences": {
      "SkipMatching": true
    }
}
```

If successful, this command returns a JSON response that contains the instance refresh ID\. 

## Skip matching and mixed instances groups<a name="skip-matching-mixed-instances-group"></a>

To update a mixed instances group, you must specify settings for a mixed instances policy in your desired configuration\. For any mixed instances policy parameters not provided in the desired configuration, Amazon EC2 Auto Scaling resets the parameter value to a default value\.

### Migrate to the default version of your launch template<a name="skip-matching-mixed-instances-group-launch-template-default"></a>

The following example shows a [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command that updates a mixed instances group to the default version of your launch template\. If there are any instances that are already using the default version of the specified launch template, they are not replaced\.

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
          "LaunchTemplateId":"lt-068f72b729example",
          "Version":"$Default"
        },
        "Overrides":[

        ... existing instance types ... 

        ]
      },
      "InstancesDistribution":{
        "OnDemandPercentageAboveBaseCapacity":50,
        "SpotAllocationStrategy":"capacity-optimized"
      }
    }
  },
  "Preferences":{
    "SkipMatching":true
  }
}
```

If successful, this command returns a JSON response that contains the instance refresh ID\. 

### Migrate to the latest version of your launch template<a name="skip-matching-mixed-instances-group-launch-template-latest"></a>

The following example shows a [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command that updates a mixed instances group to the latest version of your launch template\. If there are any instances that are already using the latest version of the specified launch template, they are not replaced\.

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
          "LaunchTemplateId":"lt-068f72b729example",
          "Version":"$Latest"
        },
        "Overrides":[

        ... existing instance types ... 

        ]
      },
      "InstancesDistribution":{
        "OnDemandPercentageAboveBaseCapacity":50,
        "SpotAllocationStrategy":"capacity-optimized"
      }
    }
  },
  "Preferences":{
    "SkipMatching":true
  }
}
```

If successful, this command returns a JSON response that contains the instance refresh ID\. 

### Migrate away from unwanted instance types<a name="skip-matching-latest-generation-instance-type"></a>

When you have a mixed instances group, you typically have a set of launch template overrides \(instance types\) that are used to provision instances\. The instance types are contained in an `Overrides` section\. To tell Amazon EC2 Auto Scaling that you want to replace instances that use a specific instance type, your desired configuration must specify the `Overrides` section without the unwanted instance type\. When an instance type in your group doesn’t match one of the instance types in the `Overrides` section, the instances are replaced as part of the instance refresh\. Note that an instance refresh does not choose the instance pools from which to provision the new instances; instead, the allocation strategies do that\.

The following example shows a [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command that updates a mixed instances group by replacing any instances that don't match an instance type specified in the desired configuration\.

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

       ... existing launch template and version ... 

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
        "OnDemandPercentageAboveBaseCapacity":50,
        "SpotAllocationStrategy":"capacity-optimized"
      }
    }
  },
  "Preferences":{
    "SkipMatching":true
  }
}
```

If successful, this command returns a JSON response that contains the instance refresh ID\. 