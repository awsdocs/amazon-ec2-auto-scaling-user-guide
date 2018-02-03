# Attach EC2 Instances to Your Auto Scaling Group<a name="attach-instance-asg"></a>

Amazon EC2 Auto Scaling provides you with an option to enable automatic scaling for one or more EC2 instances by attaching them to your existing Auto Scaling group\. After the instances are attached, they become a part of the Auto Scaling group\.

The instance that you want to attach must meet the following criteria:

+ The instance is in the `running` state\.

+ The AMI used to launch the instance must still exist\.

+ The instance is not a member of another Auto Scaling group\.

+ The instance is in the same Availability Zone as the Auto Scaling group\.

+ If the Auto Scaling group has an attached load balancer, the instance and the load balancer must both be in EC2\-Classic or the same VPC\. If the Auto Scaling group has an attached target group, the instance and the load balancer must both be in the same VPC\.

When you attach instances, the desired capacity of the group increases by the number of instances being attached\. If the number of instances being attached plus the desired capacity exceeds the maximum size of the group, the request fails\.

If you attach an instance to an Auto Scaling group that has an attached load balancer, the instance is registered with the load balancer\. If you attach an instance to an Auto Scaling group that has an attached target group, the instance is registered with the target group\.


+ [Attaching an Instance Using the AWS Management Console](#attach-instance-console)
+ [Attaching an Instance Using the AWS CLI](#attach-instance-aws-cli)

The examples use an Auto Scaling group with the following configuration:

+ Auto Scaling group name = **my\-asg**

+ Minimum size = **1**

+ Maximum size = **5**

+ Desired capacity = **2**

+ Availability Zone = **us\-west\-2a**

## Attaching an Instance Using the AWS Management Console<a name="attach-instance-console"></a>

You can attach an existing instance to an existing Auto Scaling group, or to a new Auto Scaling group as you create it\.

**To attach an instance to a new Auto Scaling group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Instances**\.

1. Select the instance\.

1. Choose **Actions**, **Instance Settings**, **Attach to Auto Scaling Group**\.

1. On the **Attach to Auto Scaling Group** page, select **a new Auto Scaling group**, type a name for the group, and then choose **Attach**\.

   The new Auto Scaling group is created using a new launch configuration with the same name that you specified for the Auto Scaling group\. The launch configuration gets its settings \(for example, security group and IAM role\) from the instance that you attached\. The Auto Scaling group gets settings \(for example, Availability Zone and subnet\) from the instance that you attached, and has a desired capacity and maximum size of `1`\.

1. \(Optional\) To edit the settings for the Auto Scaling group, on the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\. Select the new Auto Scaling group, choose **Edit**, change the settings as needed, and then choose **Save**\.

**To attach an instance to an existing Auto Scaling group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. \(Optional\) On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\. Select the Auto Scaling group and verify that the maximum size of the Auto Scaling group is large enough that you can add another instance\. Otherwise, choose **Edit**, increase the maximum size, and then choose **Save**\.

1. On the navigation pane, choose **Instances**\.

1. Select the instance\.

1. Choose **Actions**, **Instance Settings**, **Attach to Auto Scaling Group**\.

1. On the **Attach to Auto Scaling Group** page, select **an existing Auto Scaling group**, select the instance, and then choose **Attach**\.

1. If the instance doesn't meet the criteria \(for example, if it's not in the same Availability Zone as the Auto Scaling group\), you get an error message with the details\. Choose **Close** and try again with an instance that meets the criteria\.

## Attaching an Instance Using the AWS CLI<a name="attach-instance-aws-cli"></a>

**To attach an instance to an Auto Scaling group using the AWS CLI**

1. Describe a specific Auto Scaling group using the following [describe\-auto\-scaling\-groups](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command:

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names my-asg
   ```

   The following example response shows that the desired capacity is 2 and the group has 2 running instances: 

   ```
   {
       "AutoScalingGroups": [
           {
               "AutoScalingGroupARN": "arn",
               "HealthCheckGracePeriod": 300,
               "SuspendedProcesses": [],
               "DesiredCapacity": 2,
               "Tags": [],
               "EnabledMetrics": [],
               "LoadBalancerNames": [],
               "AutoScalingGroupName": "my-asg",
               "DefaultCooldown": 300,
               "MinSize": 1,
               "Instances": [
                   {
                       "InstanceId": "i-a5e87793",
                       "AvailabilityZone": "us-west-2a",
                       "HealthStatus": "Healthy",
                       "LifecycleState": "InService",
                       "LaunchConfigurationName": "my-lc"
                   },
                   {
                       "InstanceId": "i-a4e87792",
                       "AvailabilityZone": "us-west-2a",
                       "HealthStatus": "Healthy",
                       "LifecycleState": "InService",
                       "LaunchConfigurationName": "my-lc"
                   }
               ],
               "MaxSize": 5,
               "VPCZoneIdentifier": "subnet-e4f33493",
               "TerminationPolicies": [
                   "Default"
               ],
               "LaunchConfigurationName": "my-lc",
               "CreatedTime": "2014-12-12T23:30:42.611Z",
               "AvailabilityZones": [
                   "us-west-2a"
               ],
               "HealthCheckType": "EC2"
           }
       ]
   }
   ```

1. Attach an instance to the Auto Scaling group using the following [attach\-instances](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-instances.html) command:

   ```
   aws autoscaling attach-instances --instance-ids i-a8e09d9c --auto-scaling-group-name my-asg
   ```

1. To verify that the instance is attached, use the following `describe-auto-scaling-groups` command:

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names my-asg
   ```

   The following example response shows that the desired capacity has increased by 1 to 3, and that there is a new instance, `i-a8e09d9c`: 

   ```
   {
       "AutoScalingGroups": [
           {
               "AutoScalingGroupARN": "arn",
               "HealthCheckGracePeriod": 300,
               "SuspendedProcesses": [],
               "DesiredCapacity": 3,
               "Tags": [],
               "EnabledMetrics": [],
               "LoadBalancerNames": [],
               "AutoScalingGroupName": "my-asg",
               "DefaultCooldown": 300,
               "MinSize": 1,
               "Instances": [
                   {
                       "InstanceId": "i-a8e09d9c",
                       "AvailabilityZone": "us-west-2a",
                       "HealthStatus": "Healthy",
                       "LifecycleState": "InService",
                       "LaunchConfigurationName": "my-lc"
                   },
                   {
                       "InstanceId": "i-a5e87793",
                       "AvailabilityZone": "us-west-2a",
                       "HealthStatus": "Healthy",
                       "LifecycleState": "InService",
                       "LaunchConfigurationName": "my-lc"
                   },
                   {
                       "InstanceId": "i-a4e87792",
                       "AvailabilityZone": "us-west-2a",
                       "HealthStatus": "Healthy",
                       "LifecycleState": "InService",
                       "LaunchConfigurationName": "my-lc"
                   }
               ],
               "MaxSize": 5,
               "VPCZoneIdentifier": "subnet-e4f33493",
               "TerminationPolicies": [
                   "Default"
               ],
               "LaunchConfigurationName": "my-lc",
               "CreatedTime": "2014-12-12T23:30:42.611Z",
               "AvailabilityZones": [
                   "us-west-2a"
               ],
               "HealthCheckType": "EC2"
           }
       ]
   }
   ```