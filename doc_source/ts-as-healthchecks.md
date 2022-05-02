# Troubleshoot Amazon EC2 Auto Scaling: Health checks<a name="ts-as-healthchecks"></a>

This page provides information about your EC2 instances that terminate due to a health check\. It describes potential causes, and the steps that you can take to resolve the issues\. 

To retrieve an error message, see [Retrieve an error message from scaling activities](CHAP_Troubleshooting.md#RetrievingErrors)\.

**Topics**
+ [An instance was taken out of service in response to an EC2 instance status check failure](#ts-failed-status-checks)
+ [An instance was taken out of service in response to an EC2 scheduled reboot](#ts-scheduled-maintenance)
+ [An instance was taken out of service in response to an EC2 health check that indicated it had been terminated or stopped](#ts-terminated-or-stopped)
+ [An instance was taken out of service in response to an ELB system health check failure](#ts-failed-elb-health-checks)

**Note**  
You can be notified when Amazon EC2 Auto Scaling terminates the instances in your Auto Scaling group, including when the cause of instance termination is not the result of a scaling activity\. For more information, see [Get Amazon SNS notifications when your Auto Scaling group scales](ec2-auto-scaling-sns-notifications.md)\.   
The sections that follow describe the most common health check errors and causes that you'll encounter\. If you have a different issue, see the following AWS Knowledge Center articles for additional troubleshooting help:  
 [Why did Amazon EC2 Auto Scaling terminate an instance?](http://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-instance-how-terminated/) 
 [Why didn’t Amazon EC2 Auto Scaling terminate an unhealthy instance?](http://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-terminate-instance/) 

## An instance was taken out of service in response to an EC2 instance status check failure<a name="ts-failed-status-checks"></a>

**Problem**: Auto Scaling instances fail the Amazon EC2 status checks\. 

**Cause 1**: If there are issues that cause Amazon EC2 to consider the instances in your Auto Scaling group impaired, Amazon EC2 Auto Scaling automatically replaces the impaired instances as part of its health check\. Status checks are built into Amazon EC2, so they cannot be disabled or deleted\. When an instance status check fails, you typically must address the problem yourself by making instance configuration changes until your application is no longer exhibiting any problems\.

**Solution 1**: To address this issue, follow these steps:

1. Manually create an Amazon EC2 instance that is not part of the Auto Scaling group and investigate the problem\. For general help with investigating impaired instances, see [Troubleshoot instances with failed status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstances.html) in the *Amazon EC2 User Guide for Linux Instances* and [Troubleshooting Windows Instances](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/troubleshooting-windows-instances.html) in the *Amazon EC2 User Guide for Windows Instances*\. 

1. After you confirm that your instance launched successfully and is healthy, deploy a new, error\-free instance configuration to the Auto Scaling group\.

1. Delete the instance that you created to avoid ongoing charges to your AWS account\. 

**Cause 2**: There is a mismatch between the health check grace period and the instance startup time\.

**Solution 2**: Edit the health check grace period for your Auto Scaling group to an appropriate time period for your application\. Instances launched in an Auto Scaling group require sufficient warm\-up time \(grace period\) to prevent early termination due to a health check replacement\. For more information, see [Health check grace period](ec2-auto-scaling-health-checks.md#health-check-grace-period)\. 

## An instance was taken out of service in response to an EC2 scheduled reboot<a name="ts-scheduled-maintenance"></a>

**Problem**: Auto Scaling instances are replaced when a scheduled event indicates a problem with the instance\.

**Cause**: Amazon EC2 Auto Scaling replaces instances with a future scheduled maintenance or retirement event before the scheduled time\.

**Solution**: These events do not occur frequently\. If you need something to happen on the instance that is terminating, or on the instance that is starting up, you can use lifecycle hooks\. These hooks allow you to perform a custom action as Amazon EC2 Auto Scaling launches or terminates instances\. For more information, see [Amazon EC2 Auto Scaling lifecycle hooks](lifecycle-hooks.md)\. 

If you do not want instances to be replaced due to a scheduled event, you can suspend the health check process for an Auto Scaling group\. For more information, see [Suspend and resume a process for an Auto Scaling group](as-suspend-resume-processes.md)\. 

## An instance was taken out of service in response to an EC2 health check that indicated it had been terminated or stopped<a name="ts-terminated-or-stopped"></a>

**Problem**: Auto Scaling instances that have been stopped, rebooted, or terminated are replaced\. 

**Cause 1**: A user manually stopped, rebooted, or terminated the instance\.

**Solution 1**: If a health check fails because a user manually stopped, rebooted, or terminated the instance, this is due to how Amazon EC2 Auto Scaling health checks work\. The instance must be healthy and reachable\. If you need to reboot the instances in your Auto Scaling group, we recommend that you put the instances on standby first\. For more information, see [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md)\. 

Note that when you terminate instances manually, termination lifecycle hooks and Elastic Load Balancing deregistration \(and connection draining\) must be completed before the instance is actually terminated\.

**Cause 2**: Amazon EC2 Auto Scaling attempts to replace Spot Instances after the Amazon EC2 Spot service interrupts the instances, because the Spot price increases above your maximum price or capacity is no longer available\. 

**Solution 2**: There is no guarantee that a Spot Instance exists to fulfill the request at any given point in time\. However, you can try the following:
+ Use a higher Spot maximum price \(possibly the On\-Demand price\)\. By setting your maximum price higher, it gives the Amazon EC2 Spot service a better chance of launching and maintaining your required amount of capacity\.
+ Increase the number of different capacity pools that you can launch instances from by running multiple instance types in multiple Availability Zones\. For more information, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.
+ If you use multiple instance types, consider enabling the Capacity Rebalancing feature\. This is useful if you want the Amazon EC2 Spot service to attempt to launch a new Spot Instance before a running instance is terminated\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot Interruptions](ec2-auto-scaling-capacity-rebalancing.md)\.

## An instance was taken out of service in response to an ELB system health check failure<a name="ts-failed-elb-health-checks"></a>

**Problem**: Auto Scaling instances might pass the EC2 status checks\. But they might fail the Elastic Load Balancing health checks for the target groups or Classic Load Balancers with which the Auto Scaling group is registered\. 

**Cause**: If your Auto Scaling group relies on health checks provided by Elastic Load Balancing, Amazon EC2 Auto Scaling determines the health status of your instances by checking the results of both the EC2 status checks and the Elastic Load Balancing health checks\. The load balancer performs health checks by sending a request to each instance and waiting for the correct response, or by establishing a connection with the instance\. An instance might fail the Elastic Load Balancing health check because an application running on the instance has issues that cause the load balancer to consider the instance out of service\. For more information, see [Add Elastic Load Balancing health checks to an Auto Scaling group](as-add-elb-healthcheck.md)\. 

**Solution 1**: To pass the Elastic Load Balancing health checks: 
+ Make note of the success codes that the load balancer is expecting, and verify that your application is configured correctly to return these codes on success\. 
+ Verify that the security groups for your load balancer and Auto Scaling group are correctly configured\. 
+ Verify that the health check settings of your target groups are correctly configured\. You define health check settings for your load balancer per target group\. 
+ Consider adding a launch lifecycle hook to the Auto Scaling group to ensure that the applications on the instances are ready to accept traffic before they are registered to the load balancer at the end of the lifecycle hook\.
+ Set the health check grace period for your Auto Scaling group to a long enough time period to support the number of consecutive successful health checks required before Elastic Load Balancing considers a newly launched instance healthy\.
+ Verify that the load balancer is configured in the same Availability Zones as your Auto Scaling group\.

For more information, see the following topics:
+ [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html) in the *User Guide for Application Load Balancers*
+ [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html) in the *User Guide for Network Load Balancers*
+ [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/health-checks.html) in the *User Guide for Gateway Load Balancers*
+ [Configure health checks for your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-healthchecks.html) in the *User Guide for Classic Load Balancers*

**Solution 2**: Update the Auto Scaling group to disable Elastic Load Balancing health checks\.