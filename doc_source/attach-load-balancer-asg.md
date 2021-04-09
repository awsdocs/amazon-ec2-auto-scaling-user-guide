# Attaching a load balancer to your Auto Scaling group<a name="attach-load-balancer-asg"></a>

This topic describes how to attach an Elastic Load Balancing load balancer to your Auto Scaling group\. 

Amazon EC2 Auto Scaling integrates with Elastic Load Balancing to help you to insert an Application Load Balancer, Network Load Balancer, Classic Load Balancer, or Gateway Load Balancer in front of your Auto Scaling group\. Note that Classic Load Balancers are the only type of load balancer available for EC2\-Classic\. To learn more about the different types of load balancers, see [ Elastic Load Balancing types](autoscaling-load-balancer.md#integrations-aws-elastic-load-balancing-types)\.

When you attach a load balancer, it enters the `Adding` state while registering the instances in the group\. After all instances in the group are registered, it enters the `Added` state\. After at least one registered instance passes the health checks, it enters the `InService` state\. When the load balancer is in the `InService` state, Amazon EC2 Auto Scaling can terminate and replace any instances that are reported as unhealthy\. If no registered instances pass the health checks \(for example, due to a misconfigured health check\), the load balancer doesn't enter the `InService` state\. Amazon EC2 Auto Scaling doesn't terminate and replace the instances\. 

When you detach a load balancer, it enters the `Removing` state while deregistering the instances in the group\. The instances remain running after they are deregistered\. If connection draining is enabled, Elastic Load Balancing waits for in\-flight requests to complete or for the maximum timeout to expire \(whichever comes first\) before deregistering the instances\. 

**Note**  
By default, connection draining is enabled for Application Load Balancers, Network Load Balancers, and Gateway Load Balancers, but must be enabled for Classic Load Balancers\.

**Contents**
+ [Prerequisites](#as-add-load-balancer-prerequisites)
+ [Attach a load balancer \(console\)](#as-add-load-balancer-console)
  + [Attach an existing load balancer](#as-existing-load-balancer-console)
  + [Create and attach a new Application Load Balancer or Network Load Balancer](#as-create-load-balancer-console)
+ [Attach a load balancer \(AWS CLI\)](#as-add-load-balancer-aws-cli)
+ [Detach a load balancer](#as-remove-load-balancer)

## Prerequisites<a name="as-add-load-balancer-prerequisites"></a>

If necessary, create a new launch template or launch configuration for your Auto Scaling group\. Create a launch template or launch configuration with a security group that controls the traffic to and from the instances behind the load balancer\. The recommended rules depend on the type of load balancer and the types of backends that the load balancer uses\. For example, to route traffic to web servers, allow inbound HTTP access on port 80 from the load balancer\. 

Before you can deploy virtual appliances behind a Gateway Load Balancer, the launch template or launch configuration must meet the following requirements:
+ The Amazon machine image \(AMI\) must specify the ID of an AMI that supports GENEVE using a Gateway Load Balancer\.
+ The security groups must allow UDP traffic on port 6081\. 

You can configure the health check settings for a specific Auto Scaling group at any time\. For more information, see [Adding Elastic Load Balancing health checks to an Auto Scaling group](as-add-elb-healthcheck.md)\. Before adding these health checks, make sure you have a security group for your launch template or launch configuration that allows access from the load balancer on the port on which you want Elastic Load Balancing to perform health checks\. 

## Attach a load balancer \(console\)<a name="as-add-load-balancer-console"></a>

**Topics**
+ [Attach an existing load balancer](#as-existing-load-balancer-console)
+ [Create and attach a new Application Load Balancer or Network Load Balancer](#as-create-load-balancer-console)

### Attach an existing load balancer<a name="as-existing-load-balancer-console"></a>

Use the following procedures to attach a load balancer when you create or update an Auto Scaling group\. 

Start by following the procedures in the Elastic Load Balancing documentation to create a new load balancer and target group\. You can skip the step for registering your Amazon EC2 instances\. Amazon EC2 Auto Scaling automatically takes care of registering instances\. For more information, see [Getting started with Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-getting-started.html) in the *Elastic Load Balancing User Guide*\.

**To attach an existing load balancer as you are creating a new Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Choose **Create Auto Scaling group**\.

1.  In Steps 1 and 2, choose the options as desired and proceed to **Step 3: Configure advanced options**\.

1. For **Load balancing**, choose **Attach to an existing load balancer**\.

1. Under **Attach to an existing load balancer**, do one of the following:

   1. For Application Load Balancers, Network Load Balancers, and Gateway Load Balancers:

      Choose **Choose from your load balancer target groups**, and then choose a target group in the **Existing load balancer target groups** field\.

   1. For Classic Load Balancers:

      Choose **Choose from Classic Load Balancers**, and then choose your load balancer in the **Classic Load Balancers** field\.

1. Proceed to create the Auto Scaling group\. Your instances will be automatically registered to the load balancer after the Auto Scaling group has been created\. 

**To attach an existing load balancer to an existing Auto Scaling group**

Use the following procedure to attach a load balancer to an existing Auto Scaling group\. 

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Load balancing**, **Edit**\.

1. Under **Load balancing**, do one of the following:

   1. For **Application, Network or Gateway Load Balancer target groups**, select its check box and choose a target group\.

   1. For **Classic Load Balancers**, select its check box and choose your load balancer\.

1. Choose **Update**\.

### Create and attach a new Application Load Balancer or Network Load Balancer<a name="as-create-load-balancer-console"></a>

To make it easier to set up a load\-balanced application, you can use the following procedure to create and attach an Application Load Balancer or a Network Load Balancer as you create your Auto Scaling group\. 

**Important**  
After creating your Auto Scaling group, if you need to create a listener with a secure protocol, such as HTTPS, or a UDP listener, you will need to use the Elastic Load Balancing console to create additional listeners\. You can add another listener to an existing load balancer, as long as you use distinct ports\.

**To create and attach a new load balancer as you are creating a new Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1.  In Steps 1 and 2, choose the options as desired and proceed to **Step 3: Configure advanced options**\.

1. For **Load balancing**, choose **Attach to a new load balancer**\.

   1. Under **Attach to a new load balancer**, for **Load balancer type**, choose whether to create an Application Load Balancer or Network Load Balancer\. 

   1. For **Load balancer name**, enter a name for the load balancer, or keep the default name\.

   1. For **Load balancer scheme**, choose whether to create a public internet\-facing load balancer, or keep the default for an internal load balancer\.

   1. For **Availability Zones and subnets**, for each Availability Zone that you chose to launch your EC2 instances into \(which prepopulate from Step 2\), select the public subnet in those Availability Zones\.

   1. For **Listeners and routing**, update the port number for your listener \(if necessary\), and under **Default routing**, choose **Create a target group**\. Alternatively, you can choose an existing target group from the drop\-down list\.

   1. If you chose **Create a target group** in the last step, for **New target group name**, enter a name for the target group, or keep the default name\. 

   1. To add tags, choose **Add tag**, and provide a tag key and value for each tag\.

1. Proceed to create the Auto Scaling group\. Your instances will be automatically registered to the load balancer after the Auto Scaling group has been created\. 

## Attach a load balancer \(AWS CLI\)<a name="as-add-load-balancer-aws-cli"></a>

**To create an Auto Scaling group with an attached target group**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command with the `--target-group-arns` option to create an Auto Scaling group with an attached target group\. Specify the Amazon Resource Name \(ARN\) of a target group \(where you register targets by instance ID\) for an Application Load Balancer, Network Load Balancer, and Gateway Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template "LaunchTemplateName=my-launch-template,Version=1" \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --target-group-arns "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/1234567890123456" \
  --max-size 5 --min-size 1 --desired-capacity 2
```

**To attach a target group to an existing Auto Scaling group**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancer-target-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancer-target-groups.html) command to attach a target group \(where you register targets by instance ID\) to your Auto Scaling group\.

```
aws autoscaling attach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/1234567890123456"
```

**To create an Auto Scaling group with an attached Classic Load Balancer**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command with the `--load-balancer-names` option to create an Auto Scaling group with an attached Classic Load Balancer\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --load-balancer-names "my-load-balancer" \
  --max-size 5 --min-size 1 --desired-capacity 2
```

**To attach a Classic Load Balancer to an existing Auto Scaling group**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancers.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-load-balancers.html) command to attach the specified Classic Load Balancer to your Auto Scaling group\.

```
aws autoscaling attach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```

## Detach a load balancer<a name="as-remove-load-balancer"></a>

When you no longer need the load balancer, use the following procedure to detach it from your Auto Scaling group\.

**To detach a load balancer from a group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Load balancing**, **Edit**\.

1. Under **Load balancing**, do one of the following:

   1. For **Application, Network or Gateway Load Balancer target groups**, choose the delete icon \(X\) next to the target group\.

   1. For **Classic Load Balancers**, choose the delete icon \(X\) next to the load balancer\. 

1. Choose **Update**

**To detach a target group \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancer-target-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancer-target-groups.html) command to detach a target group from your Auto Scaling group if you no longer need it\. 

```
aws autoscaling detach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/1234567890123456"
```

**To detach a Classic Load Balancer \(AWS CLI\)**  
Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancers.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancers.html) command to detach a Classic Load Balancer from your Auto Scaling group if you no longer need it\.

```
aws autoscaling detach-load-balancers --auto-scaling-group-name my-asg \
  --load-balancer-names my-lb
```