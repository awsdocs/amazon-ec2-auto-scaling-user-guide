# Replacing Auto Scaling instances<a name="ec2-auto-scaling-group-replacing-instances"></a>

Amazon EC2 Auto Scaling offers capabilities that let you replace instances after you update or modify your Auto Scaling group\. Amazon EC2 Auto Scaling also helps you streamline updates by giving you the option of including them in the same operation that replaces the instances\.

This section includes information to help you do the following:
+ Start an instance refresh to replace instances in your Auto Scaling group\.
+ Declare specific updates that describe a desired configuration and update the Auto Scaling group to the desired configuration\.
+ Skip replacing already updated instances\.
+ Use checkpoints to replace instances in phases and perform verifications on your instances at specific points\.
+ Receive notifications by email when a checkpoint is reached\.
+ Limit the lifetime of instances to ensure consistent software versions and instance configurations across the Auto Scaling group\.

**Topics**
+ [Replacing instances based on an instance refresh](asg-instance-refresh.md)
+ [AWS CLI examples that enable skip matching](asg-instance-refresh-skip-matching.md)
+ [Adding checkpoints to an instance refresh](asg-adding-checkpoints-instance-refresh.md)
+ [Creating EventBridge rules for instance refresh events](monitor-events-eventbridge-sns.md)
+ [Replacing instances based on maximum instance lifetime](asg-max-instance-lifetime.md)