# Working with Amazon EC2 Auto Scaling termination policies<a name="ec2-auto-scaling-termination-policies"></a>

This topic provides detailed information about the default termination policy and the options available to you to choose different termination policies to meet your application needs\. By choosing different termination policies, you can control which instances you prefer to terminate first when a scale\-in event occurs\. For example, you can choose a different termination policy so that Amazon EC2 Auto Scaling prioritizes terminating the oldest instances first\. 

When Amazon EC2 Auto Scaling terminates instances, it attempts to maintain balance across the Availability Zones that are used by your group\. Maintaining balance across Availability Zones takes precedence over termination policies\. If one Availability Zone has more instances than the other Availability Zones that are used by the group, Amazon EC2 Auto Scaling applies the termination policies to the instances from the imbalanced Availability Zone\. If the Availability Zones used by the group are balanced, Amazon EC2 Auto Scaling applies the termination policies across all of the Availability Zones for the group\.

**Contents**
+ [Default termination policy](#default-termination-policy)
+ [Default termination policy and mixed instances groups](#default-termination-policy-mixed-instances-groups)
+ [Using different termination policies](#custom-termination-policy)
  + [Using different termination policies \(console\)](#custom-termination-policy-console)
  + [Using different termination policies \(AWS CLI\)](#custom-termination-policy-cli)

## Default termination policy<a name="default-termination-policy"></a>

The default termination policy applies multiple termination criteria before selecting an instance to terminate\. When Amazon EC2 Auto Scaling terminates instances, it first determines which Availability Zones have the most instances, and it finds at least one instance that is not protected from scale in\. Within the selected Availability Zone, the following default termination policy behavior applies:

1. Determine whether any of the instances eligible for termination use the oldest launch template or configuration:

   1. \[For Auto Scaling groups that use a launch template\]

      Determine whether any of the instances use the oldest launch template, unless there are instances that use a launch configuration\. Amazon EC2 Auto Scaling terminates instances that use a launch configuration before it terminates instances that use a launch template\.

   1. \[For Auto Scaling groups that use a launch configuration\]

      Determine whether any of the instances use the oldest launch configuration\. 

1. After applying the preceding criteria, if there are multiple unprotected instances to terminate, determine which instances are closest to the next billing hour\. If there are multiple unprotected instances closest to the next billing hour, terminate one of these instances at random\.

   Note that terminating the instance closest to the next billing hour helps you maximize the use of your instances that have an hourly charge\. Alternatively, if your Auto Scaling group uses Amazon Linux, Windows, or Ubuntu, your EC2 usage is billed in one\-second increments\. For more information, see [Amazon EC2 pricing](https://aws.amazon.com/ec2/pricing/)\.

## Default termination policy and mixed instances groups<a name="default-termination-policy-mixed-instances-groups"></a>

When an Auto Scaling group with a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md) scales in, Amazon EC2 Auto Scaling still uses termination policies to prioritize which instances to terminate, but first it identifies which of the two types \(Spot or On\-Demand\) should be terminated\. It then applies the termination policies in each Availability Zone individually\. It also identifies which instances \(within the identified purchase option\) in which Availability Zones to terminate that will result in the Availability Zones being most balanced\. The same logic applies to Auto Scaling groups that use a mixed instances configuration with weights defined for the instance types\. 

The default termination policy changes slightly due to differences in how [mixed instances policies](ec2-auto-scaling-mixed-instances-groups.md) are implemented\. The following new behavior of the default termination policy applies:

1. Determine which instances are eligible for termination in order to align the remaining instances to the [allocation strategy](ec2-auto-scaling-mixed-instances-groups.md#allocation-strategies) for the On\-Demand or Spot Instance that is terminating\.

   For example, after your instances launch, you might change the priority order of your preferred instance types\. When a scale\-in event occurs, Amazon EC2 Auto Scaling tries to gradually shift the On\-Demand Instances away from instance types that are lower priority\.

1. Determine whether any of the instances eligible for termination use the oldest launch template or configuration:

   1. \[For Auto Scaling groups that use a launch template\]

      Determine whether any of the instances use the oldest launch template, unless there are instances that use a launch configuration\. Amazon EC2 Auto Scaling terminates instances that use a launch configuration before it terminates instances that use a launch template\.

   1. \[For Auto Scaling groups that use a launch configuration\]

      Determine whether any of the instances use the oldest launch configuration\. 

1. After applying the preceding criteria, if there are multiple unprotected instances to terminate, determine which instances are closest to the next billing hour\. If there are multiple unprotected instances closest to the next billing hour, terminate one of these instances at random\.

## Using different termination policies<a name="custom-termination-policy"></a>

To specify the termination criteria to apply before Amazon EC2 Auto Scaling chooses an instance for termination, you can choose from any of the following predefined termination policies: 
+ `Default`\. Terminate instances according to the default termination policy\. This policy is useful when you want your Spot allocation strategy evaluated before any other policy, so that every time your Spot instances are terminated or replaced, you continue to make use of Spot Instances in the optimal pools\. It is also useful, for example, when you want to move off launch configurations and start using launch templates\.
+ `AllocationStrategy`\. Terminate instances in the Auto Scaling group to align the remaining instances to the allocation strategy for the type of instance that is terminating \(either a Spot Instance or an On\-Demand Instance\)\. This policy is useful when your preferred instance types have changed\. If the Spot allocation strategy is `lowest-price`, you can gradually rebalance the distribution of Spot Instances across your N lowest priced Spot pools\. If the Spot allocation strategy is `capacity-optimized`, you can gradually rebalance the distribution of Spot Instances across Spot pools where there is more available Spot capacity\. You can also gradually replace On\-Demand Instances of a lower priority type with On\-Demand Instances of a higher priority type\.
+ `OldestLaunchTemplate`\. Terminate instances that have the oldest launch template\. With this policy, instances that use the noncurrent launch template are terminated first, followed by instances that use the oldest version of the current launch template\. This policy is useful when you're updating a group and phasing out the instances from a previous configuration\.
+ `OldestLaunchConfiguration`\. Terminate instances that have the oldest launch configuration\. This policy is useful when you're updating a group and phasing out the instances from a previous configuration\.
+ `ClosestToNextInstanceHour`\. Terminate instances that are closest to the next billing hour\. This policy helps you maximize the use of your instances that have an hourly charge\. \(Only instances that use Amazon Linux, Windows, or Ubuntu are billed in one\-second increments\.\)
+ `NewestInstance`\. Terminate the newest instance in the group\. This policy is useful when you're testing a new launch configuration but don't want to keep it in production\.
+ `OldestInstance`\. Terminate the oldest instance in the group\. This option is useful when you're upgrading the instances in the Auto Scaling group to a new EC2 instance type\. You can gradually replace instances of the old type with instances of the new type\.
**Note**  
Amazon EC2 Auto Scaling always balances instances across Availability Zones first, regardless of which termination policy is used\. As a result, you might encounter situations in which some newer instances are terminated before older instances\. For example, when there is a more recently added Availability Zone, or when one Availability Zone has more instances than the other Availability Zones that are used by the group\. 

### Using different termination policies \(console\)<a name="custom-termination-policy-console"></a>

After your Auto Scaling group has been created, you can update the termination policies for your group\. The default termination policy is used automatically\. You have the option of replacing the default policy with a different termination policy \(such as `OldestLaunchTemplate`\) or multiple termination policies listed in the order in which they should apply\. 

**To choose different termination policies**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Advanced configurations**, **Edit**\.

1. For **Termination policies**, choose one or more termination policies\. If you choose multiple policies, put them in the order that you want them evaluated in\.

   You can optionally choose **Custom termination policy** and then choose a Lambda function that meets your needs\. If you have created versions and aliases for your Lambda function, you can choose a version or alias from the **Version/Alias** drop\-down\. To use the unpublished version of your Lambda function, keep **Version/Alias** set to its default\. For more information, see [Creating a custom termination policy with Lambda](lambda-custom-termination-policy.md)\.
**Note**  
When using multiple policies, their order must be set correctly:  
If you use the **Default** policy, it must be the last policy in the list\.
If you use a **Custom termination policy**, it must be the first policy in the list\.

1. Choose **Update**\.

### Using different termination policies \(AWS CLI\)<a name="custom-termination-policy-cli"></a>

The default termination policy is used automatically unless a different policy is specified\.

**To use a different termination policy**  
Use one of the following commands:
+ [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html)
+ [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html)

You can use termination policies individually, or combine them into a list of policies\. For example, use the following command to update an Auto Scaling group to use the `OldestLaunchConfiguration` policy first and then use the `ClosestToNextInstanceHour` policy\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --termination-policies "OldestLaunchConfiguration" "ClosestToNextInstanceHour"
```

If you use the `Default` termination policy, make it the last one in the list of termination policies\. For example, `--termination-policies "OldestLaunchConfiguration" "Default"`\.

To use a custom termination policy, you must first create your termination policy using AWS Lambda\. To specify the Lambda function to use as your termination policy, make it the first one in the list of termination policies\. For example, `--termination-policies "arn:aws:lambda:us-west-2:123456789012:function:HelloFunction:prod" "OldestLaunchConfiguration"`\. For more information, see [Creating a custom termination policy with Lambda](lambda-custom-termination-policy.md)\.