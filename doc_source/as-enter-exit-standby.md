# Temporarily removing instances from your Auto Scaling group<a name="as-enter-exit-standby"></a>

You can put an instance that is in the `InService` state into the `Standby` state, update or troubleshoot the instance, and then return the instance to service\. Instances that are on standby are still part of the Auto Scaling group, but they do not actively handle load balancer traffic\.

This feature helps you stop and start the instances or reboot them without worrying about Amazon EC2 Auto Scaling terminating the instances as part of its health checks or during scale\-in events\.

For example, you can change the Amazon Machine Image \(AMI\) for an Auto Scaling group at any time by changing the launch template or launch configuration\. Any subsequent instances that the Auto Scaling group launches use this AMI\. However, the Auto Scaling group does not update the instances that are currently in service\. You can terminate these instances and let Amazon EC2 Auto Scaling replace them, or use the instance refresh feature to terminate and replace the instances\. Or, you can put the instances on standby, update the software, and then put the instances back in service\.

Detaching instances from an Auto Scaling group is similar to putting instances on standby\. Detaching instances might be useful if you want to manage the instances like standalone EC2 instances and possibly terminate them\. For more information, see [Detach EC2 instances from your Auto Scaling group](detach-instance-asg.md)\.

**Important**  
When you put instances on standby, your Auto Scaling group can become unbalanced between Availability Zones\. Amazon EC2 Auto Scaling compensates by rebalancing the Availability Zones unless you suspend the `AZRebalance` process\. For more information, see [Suspending and resuming a process for an Auto Scaling group](as-suspend-resume-processes.md)\.

**Topics**
+ [How the standby state works](#standby-state)
+ [Health status of an instance in a standby state](#standby-instance-health-status)
+ [Temporarily remove an instance \(console\)](#standby-state-console)
+ [Temporarily remove an instance \(AWS CLI\)](#standby-state-aws-cli)

**Important**  
You are billed for instances that are in a standby state\.

## How the standby state works<a name="standby-state"></a>

The standby state works as follows to help you temporarily remove an instance from your Auto Scaling group:

1. You put the instance into the standby state\. The instance remains in this state until you exit the standby state\.

1. If there is a load balancer target group or Classic Load Balancer attached to your Auto Scaling group, the instance is deregistered from the load balancer\. If connection draining is enabled for the load balancer, Elastic Load Balancing waits 300 seconds by default before completing the deregistration process, which helps in\-flight requests to complete\. 

1. By default, the value that you specified as your desired capacity is decremented when you put an instance on standby\. This prevents the launch of an additional instance while you have this instance on standby\. Alternatively, you can specify that your desired capacity is not decremented\. If you specify this option, the Auto Scaling group launches an instance to replace the one on standby\. The intention is to help you maintain capacity for your application while one or more instances are on standby\. 

1. You can update or troubleshoot the instance\.

1. You return the instance to service by exiting the standby state\.

1. After you put an instance that was on standby back in service, the desired capacity is incremented\. If you did not decrement the capacity when you put the instance on standby, the Auto Scaling group detects that you have more instances than you need\. It applies the termination policy in effect to reduce the size of the group\. For more information, see [Controlling which Auto Scaling instances terminate during scale in](as-instance-termination.md)\.

1. If there is a load balancer target group or Classic Load Balancer attached to your Auto Scaling group, the instance is registered with the load balancer\.

The following illustration shows the transitions between instance states in this process:

![\[Instances enter and exit the standby state.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/standby_lifecycle.png)

For more information about the complete lifecycle of instances in an Auto Scaling group, see [Amazon EC2 Auto Scaling instance lifecycle](AutoScalingGroupLifecycle.md)\.

## Health status of an instance in a standby state<a name="standby-instance-health-status"></a>

Amazon EC2 Auto Scaling does not perform health checks on instances that are in a standby state\. While the instance is in a standby state, its health status reflects the status that it had before you put it on standby\. Amazon EC2 Auto Scaling does not perform a health check on the instance until you put it back in service\.

For example, if you put a healthy instance on standby and then terminate it, Amazon EC2 Auto Scaling continues to report the instance as healthy\. If you attempt to put a terminated instance that was on standby back in service, Amazon EC2 Auto Scaling performs a health check on the instance, determines that it is terminating and unhealthy, and launches a replacement instance\. For an introduction to health checks, see [Health checks for Auto Scaling instances](healthcheck.md)\.

## Temporarily remove an instance \(console\)<a name="standby-state-console"></a>

The following procedure demonstrates the general process for updating an instance that is currently in service\.

**To temporarily remove an instance**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Instance management** tab, in **Instances**, select an instance\.

1. Choose **Actions**, **Set to Standby**\.

1. In the **Set to Standby** dialog box, select the check box to launch a replacement instance\. Leave it unchecked to decrement the desired capacity\. Choose **Set to Standby**\.

1. You can update or troubleshoot your instance as needed\. When you have finished, continue with the next step to return the instance to service\.

1. Select the instance, choose **Actions**, **Set to InService**\. In the **Set to InService** dialog box, choose **Set to InService**\.

## Temporarily remove an instance \(AWS CLI\)<a name="standby-state-aws-cli"></a>

The following procedure demonstrates the general process for updating an instance that is currently in service\.

**To temporarily remove an instance**

1. Use the following [describe\-auto\-scaling\-instances](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-instances.html) command to identify the instance to update\.

   ```
   aws autoscaling describe-auto-scaling-instances
   ```

   The following is an example response\.

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
          ...
       ]
   }
   ```

1. Move the instance into a `Standby` state using the following [enter\-standby](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/enter-standby.html) command\. The `--should-decrement-desired-capacity` option decreases the desired capacity so that the Auto Scaling group does not launch a replacement instance\.

   ```
   aws autoscaling enter-standby --instance-ids i-05b4f7d5be44822a6 \
     --auto-scaling-group-name my-asg --should-decrement-desired-capacity
   ```

   The following is an example response\.

   ```
   {
       "Activities": [
           {
               "Description": "Moving EC2 instance to Standby: i-05b4f7d5be44822a6",
               "AutoScalingGroupName": "my-asg",
               "ActivityId": "3b1839fe-24b0-40d9-80ae-bcd883c2be32",
               "Details": "{\"Availability Zone\":\"us-west-2a\"}",
               "StartTime": "2014-12-15T21:31:26.150Z",
               "Progress": 50,
               "Cause": "At 2014-12-15T21:31:26Z instance i-05b4f7d5be44822a6 was moved to standby 
                 in response to a user request, shrinking the capacity from 4 to 3.",
               "StatusCode": "InProgress"
           }
       ]
   }
   ```

1. \(Optional\) Verify that the instance is in `Standby` using the following [describe\-auto\-scaling\-instances](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-instances.html) command\.

   ```
   aws autoscaling describe-auto-scaling-instances --instance-ids i-05b4f7d5be44822a6
   ```

   The following is an example response\. Notice that the status of the instance is now `Standby`\.

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
               "LifecycleState": "Standby"
           },
          ...
       ]
   }
   ```

1. You can update or troubleshoot your instance as needed\. When you have finished, continue with the next step to return the instance to service\.

1. Put the instance back in service using the following [exit\-standby](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/exit-standby.html) command\.

   ```
   aws autoscaling exit-standby --instance-ids i-05b4f7d5be44822a6 --auto-scaling-group-name my-asg
   ```

   The following is an example response\.

   ```
   {
       "Activities": [
           {
               "Description": "Moving EC2 instance out of Standby: i-05b4f7d5be44822a6",
               "AutoScalingGroupName": "my-asg",
               "ActivityId": "db12b166-cdcc-4c54-8aac-08c5935f8389",
               "Details": "{\"Availability Zone\":\"us-west-2a\"}",
               "StartTime": "2014-12-15T21:46:14.678Z",
               "Progress": 30,
               "Cause": "At 2014-12-15T21:46:14Z instance i-05b4f7d5be44822a6 was moved out of standby in
                  response to a user request, increasing the capacity from 3 to 4.",
               "StatusCode": "PreInService"
           }
       ]
   }
   ```

1. \(Optional\) Verify that the instance is back in service using the following `describe-auto-scaling-instances` command\.

   ```
   aws autoscaling describe-auto-scaling-instances --instance-ids i-05b4f7d5be44822a6
   ```

   The following is an example response\. Notice that the status of the instance is `InService`\.

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
          ...
       ]
   }
   ```