# Amazon EC2 Auto Scaling User Guide

-----
*****Copyright &copy; 2018 Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in 
     connection with any product or service that is not Amazon's, 
     in any manner that is likely to cause confusion among customers, 
     or in any manner that disparages or discredits Amazon. All other 
     trademarks not owned by Amazon are the property of their respective
     owners, who may or may not be affiliated with, connected to, or 
     sponsored by Amazon.

-----
## Contents
+ [What Is Amazon EC2 Auto Scaling?](what-is-amazon-ec2-auto-scaling.md)
   + [Benefits of Auto Scaling](auto-scaling-benefits.md)
   + [Auto Scaling Lifecycle](AutoScalingGroupLifecycle.md)
   + [Amazon EC2 Auto Scaling Limits](as-account-limits.md)
+ [Setting Up Amazon EC2 Auto Scaling](setting-up.md)
+ [Getting Started with Amazon EC2 Auto Scaling](GettingStartedTutorial.md)
+ [Tutorial: Set Up a Scaled and Load-Balanced Application](as-register-lbs-with-asg.md)
+ [Launch Templates](LaunchTemplates.md)
   + [Creating a Launch Template for an Auto Scaling Group](create-launch-template.md)
   + [Copying a Launch Configuration to a Launch Template](copy-launch-config.md)
   + [Replacing a Launch Configuration with a Launch Template](replace-launch-config.md)
+ [Launch Configurations](LaunchConfiguration.md)
   + [Creating a Launch Configuration](create-launch-config.md)
   + [Creating a Launch Configuration Using an EC2 Instance](create-lc-with-instanceID.md)
   + [Changing the Launch Configuration for an Auto Scaling Group](change-launch-config.md)
   + [Launching Auto Scaling Instances in a VPC](asg-in-vpc.md)
+ [Auto Scaling Groups](AutoScalingGroup.md)
   + [Creating an Auto Scaling Group Using a Launch Template](create-asg-launch-template.md)
   + [Creating an Auto Scaling Group Using a Launch Configuration](create-asg.md)
   + [Creating an Auto Scaling Group Using an EC2 Instance](create-asg-from-instance.md)
   + [Creating an Auto Scaling Group Using the Amazon EC2 Launch Wizard](create-asg-ec2-wizard.md)
   + [Tagging Auto Scaling Groups and Instances](autoscaling-tagging.md)
   + [Using a Load Balancer With an Auto Scaling Group](autoscaling-load-balancer.md)
      + [Attaching a Load Balancer to Your Auto Scaling Group](attach-load-balancer-asg.md)
      + [Using Elastic Load Balancing Health Checks with Auto Scaling](as-add-elb-healthcheck.md)
      + [Expanding Your Scaled and Load-Balanced Application to an Additional Availability Zone](as-add-availability-zone.md)
   + [Launching Spot Instances in Your Auto Scaling Group](asg-launch-spot-instances.md)
   + [Merging Your Auto Scaling Groups into a Single Multi-Zone Group](merge-auto-scaling-groups.md)
   + [Deleting Your Auto Scaling Infrastructure](as-process-shutdown.md)
+ [Scaling the Size of Your Auto Scaling Group](scaling_plan.md)
   + [Maintaining the Number of Instances in Your Auto Scaling Group](as-maintain-instance-levels.md)
   + [Manual Scaling](as-manual-scaling.md)
      + [Attach EC2 Instances to Your Auto Scaling Group](attach-instance-asg.md)
      + [Detach EC2 Instances from Your Auto Scaling Group](detach-instance-asg.md)
   + [Scheduled Scaling for Amazon EC2 Auto Scaling](schedule_time.md)
   + [Dynamic Scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)
      + [Target Tracking Scaling Policies for Amazon EC2 Auto Scaling](as-scaling-target-tracking.md)
      + [Simple and Step Scaling Policies for Amazon EC2 Auto Scaling](as-scaling-simple-step.md)
      + [Add a Scaling Policy to an Existing Auto Scaling Group](policy-updating-console.md)
      + [Scaling Based on Amazon SQS](as-using-sqs-queue.md)
   + [Scaling Cooldowns for Amazon EC2 Auto Scaling](Cooldown.md)
   + [Controlling Which Auto Scaling Instances Terminate During Scale In](as-instance-termination.md)
   + [Amazon EC2 Auto Scaling Lifecycle Hooks](lifecycle-hooks.md)
   + [Temporarily Removing Instances from Your Auto Scaling Group](as-enter-exit-standby.md)
   + [Suspending and Resuming Scaling Processes](as-suspend-resume-processes.md)
+ [Monitoring Your Auto Scaling Instances and Groups](as-monitoring-features.md)
   + [Health Checks for Auto Scaling Instances](healthcheck.md)
   + [Monitoring Your Auto Scaling Groups and Instances Using Amazon CloudWatch](as-instance-monitoring.md)
   + [Getting CloudWatch Events When Your Auto Scaling Group Scales](cloud-watch-events.md)
   + [Getting Amazon SNS Notifications When Your Auto Scaling Group Scales](ASGettingNotifications.md)
   + [Logging Amazon EC2 Auto Scaling API Calls with AWS CloudTrail](logging-using-cloudtrail.md)
+ [Controlling Access to Your Amazon EC2 Auto Scaling Resources](control-access-using-iam.md)
   + [Service-Linked Roles for Amazon EC2 Auto Scaling](autoscaling-service-linked-role.md)
   + [Required CMK Key Policy for Use with Encrypted Volumes](key-policy-requirements-EBS-encryption.md)
   + [Use an IAM Role for Applications That Run on Amazon EC2 Instances](us-iam-role.md)
+ [Troubleshooting Amazon EC2 Auto Scaling](CHAP_Troubleshooting.md)
   + [Troubleshooting Amazon EC2 Auto Scaling: EC2 Instance Launch Failures](ts-as-instancelaunchfailure.md)
   + [Troubleshooting Amazon EC2 Auto Scaling: AMI Issues](ts-as-ami.md)
   + [Troubleshooting Amazon EC2 Auto Scaling: Load Balancer Issues](ts-as-loadbalancer.md)
   + [Troubleshooting Auto Scaling: Capacity Limits](ts-as-capacity.md)
+ [Auto Scaling Resources](as-resources.md)
+ [Document History](DocumentHistory.md)