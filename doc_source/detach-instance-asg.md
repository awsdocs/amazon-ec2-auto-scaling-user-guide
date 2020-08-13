# Detach EC2 instances from your Auto Scaling group<a name="detach-instance-asg"></a>

You can remove \(detach\) an instance from an Auto Scaling group\. After the instances are detached, you can manage them independently from the rest of the Auto Scaling group\. By detaching an instance, you can:
+ Move an instance out of one Auto Scaling group and attach it to a different group\. For more information, see [Attach EC2 instances to your Auto Scaling group](attach-instance-asg.md)\.
+ Test an Auto Scaling group by creating it using existing instances running your application\. You can then detach these instances from the Auto Scaling group when your tests are complete\.

When you detach instances, you have the option of decrementing the desired capacity for the Auto Scaling group by the number of instances that you are detaching\. If you choose not to decrement the capacity, Amazon EC2 Auto Scaling launches new instances to replace the ones that you detach\. If you decrement the capacity but detach multiple instances from the same Availability Zone, Amazon EC2 Auto Scaling can rebalance the Availability Zones unless you suspend the `AZRebalance` process\. For more information, see [Scaling processes](as-suspend-resume-processes.md#process-types)\.

If the number of instances that you are detaching decreases the size of the Auto Scaling group below its minimum capacity, you must decrement the minimum capacity for the group before you can detach the instances\.

If you detach an instance from an Auto Scaling group that has an attached load balancer, the instance is deregistered from the load balancer\. If you detach an instance from an Auto Scaling group that has an attached target group, the instance is deregistered from the target group\. If connection draining is enabled for your load balancer, Amazon EC2 Auto Scaling waits for in\-flight requests to complete\.

The examples use an Auto Scaling group with the following configuration:
+ Auto Scaling group name = `my-asg`
+ Minimum size = `1`
+ Maximum size = `5`
+ Desired capacity = `4`
+ Availability Zone = `us-west-2a`

## Detaching instances \(console\)<a name="detach-instance-console"></a>

Use the following procedure to detach an instance from your Auto Scaling group\.

**To detach an instance from an existing Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Instance management** tab, in **Instances**, select an instance and choose **Actions**, **Detach**\. \(Old console: The **Instances** tab is where you can detach the instance\.\) 

1. In the **Detach instance** dialog box, select the check box to launch a replacement instance, or leave it unchecked to decrement the desired capacity\. Choose **Detach instance**\.

## Detaching instances \(AWS CLI\)<a name="detach-instance-aws-cli"></a>

Use the following procedure to detach an instance from your Auto Scaling group\.

**To detach an instance from an existing Auto Scaling group**

1. List the current instances using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-instances.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-instances.html) command\.

   ```
   aws autoscaling describe-auto-scaling-instances
   ```

   The following example response shows that the group has four running instances\.

   ```
   {
       "AutoScalingInstances": [
           {
               "ProtectedFromScaleIn": false,
               "AvailabilityZone": "us-west-2a",
               "LaunchTemplate": {
                   "LaunchTemplateName": "my-launch-template",
                   "Version": "1",
                   "LaunchTemplateId": "lt-050555ad16a3f9c7f"
               },
               "InstanceId": "i-05b4f7d5be44822a6",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService"
           },
           {
               "ProtectedFromScaleIn": false,
               "AvailabilityZone": "us-west-2a",
               "LaunchTemplate": {
                   "LaunchTemplateName": "my-launch-template",
                   "Version": "1",
                   "LaunchTemplateId": "lt-050555ad16a3f9c7f"
               },
               "InstanceId": "i-0c20ac468fa3049e8",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService"
           },
           {
               "ProtectedFromScaleIn": false,
               "AvailabilityZone": "us-west-2a",
               "LaunchTemplate": {
                   "LaunchTemplateName": "my-launch-template",
                   "Version": "1",
                   "LaunchTemplateId": "lt-050555ad16a3f9c7f"
               },
               "InstanceId": "i-0787762faf1c28619",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService"
           },
           {
               "ProtectedFromScaleIn": false,
               "AvailabilityZone": "us-west-2a",
               "LaunchTemplate": {
                   "LaunchTemplateName": "my-launch-template",
                   "Version": "1",
                   "LaunchTemplateId": "lt-050555ad16a3f9c7f"
               },
               "InstanceId": "i-0f280a4c58d319a8a",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService"
           }
       ]
   }
   ```

1. Detach an instance and decrement the desired capacity using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-instances.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-instances.html) command\.

   ```
   aws autoscaling detach-instances --instance-ids i-05b4f7d5be44822a6 \
     --auto-scaling-group-name my-asg --should-decrement-desired-capacity
   ```

1. Verify that the instance is detached using the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-instances.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-instances.html) command\.

   ```
   aws autoscaling describe-auto-scaling-instances
   ```

   The following example response shows that there are now three running instances\. 

   ```
   {
       "AutoScalingInstances": [
           {
               "ProtectedFromScaleIn": false,
               "AvailabilityZone": "us-west-2a",
               "LaunchTemplate": {
                   "LaunchTemplateName": "my-launch-template",
                   "Version": "1",
                   "LaunchTemplateId": "lt-050555ad16a3f9c7f"
               },
               "InstanceId": "i-0c20ac468fa3049e8",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService"
           },
           {
               "ProtectedFromScaleIn": false,
               "AvailabilityZone": "us-west-2a",
               "LaunchTemplate": {
                   "LaunchTemplateName": "my-launch-template",
                   "Version": "1",
                   "LaunchTemplateId": "lt-050555ad16a3f9c7f"
               },
               "InstanceId": "i-0787762faf1c28619",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService"
           },
           {
               "ProtectedFromScaleIn": false,
               "AvailabilityZone": "us-west-2a",
               "LaunchTemplate": {
                   "LaunchTemplateName": "my-launch-template",
                   "Version": "1",
                   "LaunchTemplateId": "lt-050555ad16a3f9c7f"
               },
               "InstanceId": "i-0f280a4c58d319a8a",
               "AutoScalingGroupName": "my-asg",
               "HealthStatus": "HEALTHY",
               "LifecycleState": "InService"
           }
       ]
   }
   ```