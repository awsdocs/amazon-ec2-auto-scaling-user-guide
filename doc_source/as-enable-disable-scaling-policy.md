# Disabling a scaling policy for an Auto Scaling group<a name="as-enable-disable-scaling-policy"></a>

This topic describes how to temporarily disable a scaling policy so it won't initiate changes to the number of instances the Auto Scaling group contains\. When you disable a scaling policy, the configuration details are preserved, so you can quickly re\-enable the policy\. This is easier than temporarily deleting a policy when you don't need it, and recreating it later\. 

When a scaling policy is disabled, the Auto Scaling group does not scale out or scale in for the metric alarms that are breached while the scaling policy is disabled\. However, any scaling activities still in progress are not stopped\. 

Note that disabled scaling policies still count toward your quotas on the number of scaling policies that you can add to an Auto Scaling group\. 

**To disable a scaling policy \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. The original console is open by default\. To access the new console, on the banner at the top of the page, choose **Go to the new console**\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Automatic scaling** tab, in **Scaling policies**, select a scaling policy, and then choose **Actions**, **Disable**\.

When you are ready to re\-enable the scaling policy, repeat these steps and then choose **Actions**, **Enable**\. After you re\-enable a scaling policy, your Auto Scaling group may immediately initiate a scaling action if there are any alarms currently in ALARM state\.

**To disable a scaling policy \(AWS CLI\)**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command with the `--no-enabled` option as follows\. Specify all options in the command as you would specify them when creating the policy\.

```
aws autoscaling put-scaling-policy --auto-scaling-group-name my-asg \
   --policy-name my-scaling-policy --policy-type TargetTrackingScaling \
   --estimated-instance-warmup 360 \
   --target-tracking-configuration '{ "TargetValue": 70, "PredefinedMetricSpecification": { "PredefinedMetricType": "ASGAverageCPUUtilization" } }' \ 
   --no-enabled
```

**To re\-enable a scaling policy \(AWS CLI\)**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/put-scaling-policy.html) command with the `--enabled` option as follows\. Specify all options in the command as you would specify them when creating the policy\.

```
aws autoscaling put-scaling-policy --auto-scaling-group-name my-asg \
   --policy-name my-scaling-policy --policy-type TargetTrackingScaling \
   --estimated-instance-warmup 360 \
   --target-tracking-configuration '{ "TargetValue": 70, "PredefinedMetricSpecification": { "PredefinedMetricType": "ASGAverageCPUUtilization" } }' \ 
   --enabled
```

**To describe a scaling policy \(AWS CLI\)**  
Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-policies.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-policies.html) command to verify the enabled status of a scaling policy\.

```
aws autoscaling describe-policies --auto-scaling-group-name my-asg \
   --policy-names my-scaling-policy
```

The following is example output\.

```
{
    "ScalingPolicies": [
        {
            "AutoScalingGroupName": "my-asg",
            "PolicyName": "my-scaling-policy",
            "PolicyARN": "arn:aws:autoscaling:us-west-2:123456789012:scalingPolicy:1d52783a-b03b-4710-bb0e-549fd64378cc:autoScalingGroupName/my-asg:policyName/my-scaling-policy",
            "PolicyType": "TargetTrackingScaling",
            "StepAdjustments": [],
            "Alarms": [
                {
                    "AlarmName": "TargetTracking-my-asg-AlarmHigh-9ca53fdd-7cf5-4223-938a-ae1199204502",
                    "AlarmARN": "arn:aws:cloudwatch:us-west-2:123456789012:alarm:TargetTracking-my-asg-AlarmHigh-9ca53fdd-7cf5-4223-938a-ae1199204502"
                },
                {
                    "AlarmName": "TargetTracking-my-asg-AlarmLow-7010c83d-d55a-4a7a-abe0-1cf8b9de6d6c",
                    "AlarmARN": "arn:aws:cloudwatch:us-west-2:123456789012:alarm:TargetTracking-my-asg-AlarmLow-7010c83d-d55a-4a7a-abe0-1cf8b9de6d6c"
                }
            ],
            "TargetTrackingConfiguration": {
                "PredefinedMetricSpecification": {
                    "PredefinedMetricType": "ASGAverageCPUUtilization"
                },
                "TargetValue": 70.0,
                "DisableScaleIn": false
            },
            "Enabled": true
        }
    ]
}
```