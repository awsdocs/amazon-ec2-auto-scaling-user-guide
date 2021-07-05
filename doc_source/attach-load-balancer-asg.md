# Attaching a load balancer to your Auto Scaling group<a name="attach-load-balancer-asg"></a>

This topic describes how to attach an Elastic Load Balancing load balancer to your Auto Scaling group\. Amazon EC2 Auto Scaling integrates with Elastic Load Balancing to help you to insert an Application Load Balancer, Network Load Balancer, Classic Load Balancer, or Gateway Load Balancer in front of your Auto Scaling group\. Classic Load Balancers are the only type of load balancer available for EC2\-Classic\. To learn more about the different types of load balancers, see [ Elastic Load Balancing types](autoscaling-load-balancer.md#integrations-aws-elastic-load-balancing-types)\.

When you attach an Application Load Balancer, Network Load Balancer, or Gateway Load Balancer, you attach a target group\. Amazon EC2 Auto Scaling adds instances to the attached target group when they are launched\. You can attach one or multiple target groups, and configure health checks on a per target group basis\. 

**Contents**
+ [Understand load balancer status](#load-balancer-status)
+ [Attach an existing load balancer](#as-add-load-balancer-console)
+ [Configure an Application Load Balancer or Network Load Balancer using the Amazon EC2 Auto Scaling console](#as-create-load-balancer-console)
+ [Detach a load balancer](#as-remove-load-balancer)

## Understand load balancer status<a name="load-balancer-status"></a>

When you attach a load balancer, it enters the `Adding` state while registering the instances in the group\. After all instances in the group are registered, it enters the `Added` state\. After at least one registered instance passes the health checks, it enters the `InService` state\. When the load balancer is in the `InService` state, Amazon EC2 Auto Scaling can terminate and replace any instances that are reported as unhealthy\. If no registered instances pass the health checks \(for example, due to a misconfigured health check\), the load balancer doesn't enter the `InService` state\. Amazon EC2 Auto Scaling doesn't terminate and replace the instances\. 

When you detach a load balancer, it enters the `Removing` state while deregistering the instances in the group\. The instances remain running after they are deregistered\. By default, connection draining is enabled for Application Load Balancers, Network Load Balancers, and Gateway Load Balancers\. If connection draining is enabled, Elastic Load Balancing waits for in\-flight requests to complete or for the maximum timeout to expire \(whichever comes first\) before it deregisters the instances\. 

## Attach an existing load balancer<a name="as-add-load-balancer-console"></a>

You can attach an existing load balancer to an Auto Scaling group when you create or update the group\. If you want to create and attach a new Application Load Balancer or Network Load Balancer at the same time as you create the group, see [Configure an Application Load Balancer or Network Load Balancer using the Amazon EC2 Auto Scaling console](#as-create-load-balancer-console)\. 

**To attach an existing load balancer as you are creating a new Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Choose **Create Auto Scaling group**\.

1.  In steps 1 and 2, choose the options as desired and proceed to **Step 3: Configure advanced options**\.

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

## Configure an Application Load Balancer or Network Load Balancer using the Amazon EC2 Auto Scaling console<a name="as-create-load-balancer-console"></a>

Use the following procedure to create and attach an Application Load Balancer or a Network Load Balancer as you create your Auto Scaling group\. 

**To create and attach a new load balancer as you create a new Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Choose **Create Auto Scaling group**\.

1.  In steps 1 and 2, choose the options as desired and proceed to **Step 3: Configure advanced options**\.

1. For **Load balancing**, choose **Attach to a new load balancer**\.

   1. Under **Attach to a new load balancer**, for **Load balancer type**, choose whether to create an Application Load Balancer or Network Load Balancer\. 

   1. For **Load balancer name**, enter a name for the load balancer, or keep the default name\.

   1. For **Load balancer scheme**, choose whether to create a public internet\-facing load balancer, or keep the default for an internal load balancer\.

   1. For **Availability Zones and subnets**, select the public subnet for each Availability Zone in which you chose to launch your EC2 instances\. \(These prepopulate from step 2\.\)\.

   1. For **Listeners and routing**, update the port number for your listener \(if necessary\), and under **Default routing**, choose **Create a target group**\. Alternatively, you can choose an existing target group from the drop\-down list\.

   1. If you chose **Create a target group** in the last step, for **New target group name**, enter a name for the target group, or keep the default name\. 

   1. To add tags, choose **Add tag**, and provide a tag key and value for each tag\.

1. Proceed to create the Auto Scaling group\. Your instances will be automatically registered to the load balancer after the Auto Scaling group has been created\. 
**Note**  
After creating your Auto Scaling group, you can use the Elastic Load Balancing console to create additional listeners\. This is useful if you need to create a listener with a secure protocol, such as HTTPS, or a UDP listener\. You can add more listeners to existing load balancers, as long as you use distinct ports\.

## Detach a load balancer<a name="as-remove-load-balancer"></a>

When you no longer need the load balancer, use the following procedure to detach it from your Auto Scaling group\.

**To detach a load balancer from a group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **AUTO SCALING**, choose **Auto Scaling Groups**\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Load balancing**, **Edit**\.

1. Under **Load balancing**, do one of the following:

   1. For **Application, Network or Gateway Load Balancer target groups**, choose the delete \(X\) icon next to the target group\.

   1. For **Classic Load Balancers**, choose the delete \(X\) icon next to the load balancer\. 

1. Choose **Update**