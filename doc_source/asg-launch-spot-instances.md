# Launching Spot Instances in Your Auto Scaling Group<a name="asg-launch-spot-instances"></a>

Spot Instances are a cost\-effective choice compared to On\-Demand instances, if you can be flexible about when your applications run and if your applications can be interrupted\. You can configure your Auto Scaling group to launch Spot Instances instead of On\-Demand instances\.

Before launching Spot Instances using Amazon EC2 Auto Scaling, we recommend that you become familiar with launching and managing Spot Instances using Amazon EC2\. For more information, see [Spot Instances](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html) in the *Amazon EC2 User Guide for Linux Instances*\.

Here's how Spot Instances work with Amazon EC2 Auto Scaling:

+ **Setting your maximum price\.** You can set the maximum price you are willing to pay in the launch configuration or launch template\. You can't launch both On\-Demand instances and Spot Instances\.

+ **Changing your maximum price\.** You must create a launch configuration or launch template with the new price\. With a new launch configuration, you must associate it with your Auto Scaling group\. With a launch template, you can configure the Auto Scaling group to use the default template or the latest version of the template, so it will automatically be associated with the Auto Scaling group\. Note that the existing instances continue to run as long as the maximum price specified in the launch configuration or launch template used for those instances is higher than the current Spot market price\.

+ **Spot market price and your maximum price\.** If the market price for Spot Instances rises above your maximum price for a running instance in your Auto Scaling group, Amazon EC2 terminates your instance\. If your maximum price exactly matches the Spot market price, whether your request is fulfilled depends on several factors—such as available Spot Instance capacity\.

+ **Maintaining your Spot Instances\.** When your Spot Instance is terminated, the Auto Scaling group attempts to launch a replacement instance to maintain the desired capacity for the group\. If your maximum price is higher than the Spot market price, then it launches a Spot Instance\. Otherwise, it keeps trying\.

+ **Balancing across Availability Zones\.** If you specify multiple Availability Zones, Auto Scaling distributes the Spot requests across these Availability Zones\. If your maximum price is too low in one Availability Zone for any requests to be fulfilled, Amazon EC2 Auto Scaling checks whether requests were fulfilled in the other Availability Zones\. If so, Amazon EC2 Auto Scaling cancels the requests that failed and redistributes them across the Availability Zones that have requests fulfilled\. If the price in an Availability Zone with no fulfilled requests drops enough that future requests succeed, Auto Scaling rebalances across all the Availability Zones\. For more information, see [Rebalancing Activities](auto-scaling-benefits.md#AutoScalingBehavior.InstanceUsage)\.

+ **Spot Instance termination\.** Amazon EC2 Auto Scaling can terminate or replace Spot Instances just as it can terminate or replace On\-Demand instances\. For more information, see [Controlling Which Auto Scaling Instances Terminate During Scale In](as-instance-termination.md)\.