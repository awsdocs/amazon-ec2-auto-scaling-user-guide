# Attaching a load balancer to your Auto Scaling group<a name="attach-load-balancer-asg"></a>

This topic describes how to attach your Elastic Load Balancing load balancer to an existing Auto Scaling group\. To attach your load balancer to your Auto Scaling group when you create the group, see [Tutorial: Set up a scaled and load\-balanced application](as-register-lbs-with-asg.md)\. 

Amazon EC2 Auto Scaling integrates with Elastic Load Balancing to enable you to insert one or more Classic Load Balancers or a single Application Load Balancer or Network Load Balancer with multiple target groups in front of your Auto Scaling group\. To learn more about the different types of load balancers, see [ Elastic Load Balancing types](autoscaling-load-balancer.md#integrations-aws-elastic-load-balancing-types)\. 

When you attach a load balancer, it enters the `Adding` state while registering the instances in the group\. After all instances in the group are registered with the load balancer, it enters the `Added` state\. After at least one registered instance passes the health checks, it enters the `InService` state\. After the load balancer enters the `InService` state, Amazon EC2 Auto Scaling can terminate and replace any instances that are reported as unhealthy\. If no registered instances pass the health checks \(for example, due to a misconfigured health check\), the load balancer doesn't enter the `InService` state\. Amazon EC2 Auto Scaling doesn't terminate and replace the instances\.

When you detach a load balancer, it enters the `Removing` state while deregistering the instances in the group\. The instances remain running after they are deregistered\. If connection draining is enabled, Elastic Load Balancing waits for in\-flight requests to complete or for the maximum timeout to expire \(whichever comes first\) before deregistering the instances\. By default, connection draining is enabled for Application Load Balancers but must be enabled for Classic Load Balancers\. For more information, see [Connection draining](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/config-conn-drain.html) in the *User Guide for Classic Load Balancers*\.

**Topics**
+ [Prerequisites](#as-add-load-balancer-prerequisites)
+ [Attach a load balancer \(console\)](#as-add-load-balancer-console)
+ [Attach a load balancer \(AWS CLI\)](#as-add-load-balancer-aws-cli)

## Prerequisites<a name="as-add-load-balancer-prerequisites"></a>

Before you begin, create an Application Load Balancer or Network Load Balancer in the same AWS Region as the Auto Scaling group\. We recommend the new load balancers, but you can still use a Classic Load Balancer if it supports the features you're looking for\. 

\(Optional\) To configure your Auto Scaling group to use Elastic Load Balancing health checks, see [Adding Elastic Load Balancing health checks to an Auto Scaling group](as-add-elb-healthcheck.md)\.

Amazon EC2 Auto Scaling has changed the user interface\. By default, you're shown the new user interface, but you can choose to return to the old user interface\. This topic contains steps for each\. 

## Attach a load balancer \(console\)<a name="as-add-load-balancer-console"></a>

Use the following procedure to attach a load balancer to an existing Auto Scaling group\. 

**To attach a load balancer to a group \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Load balancing**, **Edit**\.

1. Under **Load balancing**, do one of the following:

   1. \[Application/Network Load Balancers\] For **Choose a target group for your load balancer**, choose your target group\.

   1. \[Classic Load Balancers\] For **Choose a load balancer**, choose your load balancer\.

1. Choose **Update**\.

**To attach a load balancer to a group \(old console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Choose an existing group from the list\.

1. On the **Details** tab, choose **Edit**\.

1. Do one of the following:

   1. \[Classic Load Balancers\] For **Classic Load Balancers**, choose your load balancer\.

   1. \[Application/Network Load Balancers\] For **Target Groups**, choose your target group\.

1. Choose **Save**\.

When you no longer need the load balancer, use the following procedure to detach it from your Auto Scaling group\.

**To detach a load balancer from a group \(new console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Load balancing**, **Edit**\.

1. Under **Load balancing**, do one of the following:

   1. \[Application/Network Load Balancers\] For **Choose a target group for your load balancer**, choose the delete icon \(X\) next to the target group\.

   1. \[Classic Load Balancers\] For **Choose a load balancer**, choose the delete icon \(X\) next to the load balancer\. 

1. Choose **Update**\. 

**To detach a load balancer from a group \(old console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Choose an existing group from the list\.

1. On the **Details** tab, choose **Edit**\.

1. Do one of the following:

   1. \[Classic Load Balancers\] For **Classic Load Balancers**, remove the load balancer\.

   1. \[Application/Network Load Balancers\] For **Target Groups**, remove the target group\.

1. Choose **Save**\.

## Attach a load balancer \(AWS CLI\)<a name="as-add-load-balancer-aws-cli"></a>

**To attach a target group for an Application Load Balancer or Network Load Balancer**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancer-target-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancer-target-groups.html) command to attach the specified target group to your Auto Scaling group\.

```
aws autoscaling attach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns my-targetgroup-arn
```

**To detach a target group for an Application Load Balancer or Network Load Balancer**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancer-target-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancer-target-groups.html) command to detach a target group from your Auto Scaling group if you no longer need it\.

```
aws autoscaling detach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns my-targetgroup-arn
```

**To attach a Classic Load Balancer**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancers.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancers.html) command to attach the specified load balancer to your Auto Scaling group\.

```
aws autoscaling attach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```

**To detach a Classic Load Balancer**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancers.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancers.html) command to detach a load balancer from your Auto Scaling group if you no longer need it\.

```
aws autoscaling detach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```