# Instance refresh examples that enable skip matching with the AWS Command Line Interface \(AWS CLI\)<a name="asg-instance-refresh-skip-matching"></a>

This section includes AWS CLI examples for starting an instance refresh with skip matching enabled\. Skip matching tells Amazon EC2 Auto Scaling to ignore instances that already have your latest updates, so that you don't replace more instances than you need to\. 

A common use case for skip matching is when you want to make sure your Auto Scaling group uses a particular version of your launch template and only replaces those instances that don't\.

The following considerations apply to skip matching:
+ If you enable skip matching and specify a *desired configuration*, Amazon EC2 Auto Scaling doesn't replace instances that match your desired configuration\. When the instance refresh succeeds, Amazon EC2 Auto Scaling updates the group to reflect your desired configuration\. If you enable skip matching, but you don't specify a desired configuration, then Amazon EC2 Auto Scaling doesn't replace instances that have the same configuration options that your Auto Scaling group was using before the start of the instance refresh\.
+ You can use skip matching with a new launch template, a new version of the current launch template, or a set of instance types for a mixed instances group\. If you enable skip matching, but none of these are changing, the instance refresh will succeed immediately without replacing any instances\. 
+ You cannot use skip matching with a new launch configuration\.

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