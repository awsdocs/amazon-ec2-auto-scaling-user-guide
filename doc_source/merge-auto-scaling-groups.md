# Merging your Auto Scaling groups into a single multi\-zone group<a name="merge-auto-scaling-groups"></a>

To merge separate single\-zone Auto Scaling groups into a single group spanning multiple Availability Zones, rezone one of the single\-zone groups into a multi\-zone group\. Then, delete the other groups\. This works for groups with or without a load balancer, as long as the new multi\-zone group is in one of the same Availability Zones as the original single\-zone groups\.

The following examples assume that you have two identical groups in two different Availability Zones, `us-west-2a` and `us-west-2c`\. These two groups share the following specifications: 
+ Minimum size = 2
+ Maximum size = 5
+ Desired capacity = 3

## Merge zones \(AWS CLI\)<a name="as-merge-groups-aws-cli"></a>

Use the following procedure to merge `my-group-a` and `my-group-c` into a single group that covers both `us-west-2a` and `us-west-2c`\.

**To merge separate single\-zone groups into a single multi\-zone group**

1. Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to add the `us-west-2c` Availability Zone to the supported Availability Zones for `my-group-a`\. Increase the maximum size of this group to allow for the instances from both single\-zone groups\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-group-a \
     --availability-zones "us-west-2a" "us-west-2c" \
     –-max-size 10 –-min-size 4
   ```

1. Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/set-desired-capacity.html) command to increase the size of `my-group-a`\.

   ```
   aws autoscaling set-desired-capacity --auto-scaling-group-name my-group-a \
     --desired-capacity 6
   ```

1. \(Optional\) Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that `my-group-a` is at its new size\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-group-a
   ```

1. Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to remove the instances from `my-group-c`\.

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-group-c \
     --min-size 0 --max-size 0
   ```

1. \(Optional\) Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that no instances remain in `my-group-c`\.

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-group-c
   ```

   The following is example output\.

   ```
   {
       "AutoScalingGroups": [
           {
               "AutoScalingGroupARN": "arn",
               "HealthCheckGracePeriod": 300,
               "SuspendedProcesses": [],
               "DesiredCapacity": 0,
               "Tags": [],
               "EnabledMetrics": [],
               "LoadBalancerNames": [],
               "AutoScalingGroupName": "my-group-c",
               "DefaultCooldown": 300,
               "MinSize": 0,
               "Instances": [],
               "MaxSize": 0,
               "VPCZoneIdentifier": "null",
               "TerminationPolicies": [
                   "Default"
               ],
               "LaunchConfigurationName": "my-launch-config",
               "CreatedTime": "2015-02-26T18:24:14.449Z",
               "AvailabilityZones": [
                   "us-west-2c"
               ],
               "HealthCheckType": "EC2"
           }
       ]
   }
   ```

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-auto-scaling-group.html) command to delete `my-group-c`\.

   ```
   aws autoscaling delete-auto-scaling-group --auto-scaling-group-name my-group-c
   ```