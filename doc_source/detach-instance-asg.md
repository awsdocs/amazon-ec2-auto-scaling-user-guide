# Detach EC2 Instances from Your Auto Scaling Group<a name="detach-instance-asg"></a>

You can remove an instance from an Auto Scaling group\. After the instances are detached, you can manage them independently from the rest of the Auto Scaling group\. By detaching an instance, you can:
+ Move an instance out of one Auto Scaling group and attach it to a different one\. For more information, see [Attach EC2 Instances to Your Auto Scaling Group](attach-instance-asg.md)\.
+ Test an Auto Scaling group by creating it using existing instances running your application, and then detach these instances from the Auto Scaling group when your tests are complete\.

When you detach instances, you have the option of decrementing the desired capacity for the Auto Scaling group by the number of instances being detached\. If you choose not to decrement the capacity, Amazon EC2 Auto Scaling launches new instances to replace the ones that you detached\. If you decrement the capacity but detach multiple instances from the same Availability Zone, Amazon EC2 Auto Scaling can rebalance the Availability Zones unless you suspend the `AZRebalance` process\. For more information, see [Scaling Processes](as-suspend-resume-processes.md#process-types)\.

If the number of instances that you are detaching would drop the size of the Auto Scaling group below its minimum capacity, you must decrement the minimum capacity for the Auto Scaling group before you can detach the instances\.

If you detach an instance from an Auto Scaling group that has an attached load balancer, the instance is deregistered from the load balancer\. If you detach an instance from an Auto Scaling group that has an attached target group, the instance is deregistered from the target group\. If connection draining is enabled for your load balancer, Amazon EC2 Auto Scaling waits for in\-flight requests to complete\.

**Topics**
+ [Detaching Instances Using the AWS Management Console](#detach-instance-console)
+ [Detaching Instances Using the AWS CLI](#detach-instance-aws-cli)

The examples use an Auto Scaling group with the following configuration:
+ Auto Scaling group name = `my-asg`
+ Minimum size = `1`
+ Maximum size = `5`
+ Desired capacity = `4`
+ Availability Zone = `us-west-2a`

## Detaching Instances Using the AWS Management Console<a name="detach-instance-console"></a>

Use the following procedure to detach an instance from your Auto Scaling group\.

**To detach an instance from an existing Auto Scaling group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Instances** tab, select the instance and choose **Actions**, **Detach**\.

1. On the **Detach Instance** page, select the check box to launch a replacement instance, or leave it unchecked to decrement the desired capacity\. Choose **Detach Instance**\.

## Detaching Instances Using the AWS CLI<a name="detach-instance-aws-cli"></a>

Use the following procedure to detach an instance from your Auto Scaling group\.

**To detach an instance from an existing Auto Scaling group using the AWS CLI**

1. List the current instances using the following [describe\-auto\-scaling\-instances](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-instances.html) command:

   ```
   aws autoscaling describe-auto-scaling-instances
   ```

   The following example response shows that the group has 4 running instances:

   ```
   {
       "AutoScalingInstances": [
           {
               "AvailabilityZone": "us-west-2a",
               "InstanceId": "i-2a2d8978",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService",
               "LaunchConfigurationName": "my-lc"
           },
           {
               "AvailabilityZone": "us-west-2a",
               "InstanceId": "i-5f2e8a0d",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService",
               "LaunchConfigurationName": "my-lc"
           }
           {
               "AvailabilityZone": "us-west-2a",
               "InstanceId": "i-a52387f7",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService",
               "LaunchConfigurationName": "my-lc"
           }
           {
               "AvailabilityZone": "us-west-2a",
               "InstanceId": "i-f42d89a6",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService",
               "LaunchConfigurationName": "my-lc"
           }
       ]
   }
   ```

1. Detach an instance and decrement the desired capacity using the following [detach\-instances](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-instances.html) command:

   ```
   aws autoscaling detach-instances --instance-ids i-2a2d8978 --auto-scaling-group-name my-asg --should-decrement-desired-capacity
   ```

1. Verify that the instance is detached using the following `describe-auto-scaling-instances` command:

   ```
   aws autoscaling describe-auto-scaling-instances
   ```

   The following example response shows that there are now 3 running instances: 

   ```
   {
       "AutoScalingInstances": [
           {
               "AvailabilityZone": "us-west-2a",
               "InstanceId": "i-5f2e8a0d",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService",
               "LaunchConfigurationName": "my-lc"
           }
           {
               "AvailabilityZone": "us-west-2a",
               "InstanceId": "i-a52387f7",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService",
               "LaunchConfigurationName": "my-lc"
           }
           {
               "AvailabilityZone": "us-west-2a",
               "InstanceId": "i-f42d89a6",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService",
               "LaunchConfigurationName": "my-lc"
           }
       ]
   }
   ```