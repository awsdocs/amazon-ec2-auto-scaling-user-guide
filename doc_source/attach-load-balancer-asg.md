# Attach a load balancer to your Auto Scaling group<a name="attach-load-balancer-asg"></a>

This topic describes how to attach an Elastic Load Balancing load balancer to your Auto Scaling group\. Amazon EC2 Auto Scaling integrates with Elastic Load Balancing to help you to insert an Application Load Balancer, Network Load Balancer, Classic Load Balancer, or Gateway Load Balancer in front of your Auto Scaling group\. To learn more about the different types of load balancers, see [Elastic Load Balancing types](autoscaling-load-balancer.md#integrations-aws-elastic-load-balancing-types)\.

When you attach an Application Load Balancer, Network Load Balancer, or Gateway Load Balancer, you attach a target group\. Amazon EC2 Auto Scaling adds instances to the attached target group when they are launched\. You can attach one or multiple target groups, and configure health checks on a per target group basis\. 

For an introductory guide for attaching a target group to your Auto Scaling group, see [Tutorial: Set up a scaled and load\-balanced application](tutorial-ec2-auto-scaling-load-balancer.md)\.

**Contents**
+ [Attach an existing load balancer](#as-add-load-balancer-console)
+ [Detach a load balancer](#as-remove-load-balancer)

## Attach an existing load balancer<a name="as-add-load-balancer-console"></a>

You can attach an existing load balancer to an Auto Scaling group when you create or update the group\. If you want to create and attach a new Application Load Balancer or Network Load Balancer at the same time that you create the group, see [Configure an Application Load Balancer or Network Load Balancer from the Amazon EC2 Auto Scaling console](as-create-load-balancer-console.md)\. 

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

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Load balancing**, **Edit**\.

1. Under **Load balancing**, do one of the following:

   1. For **Application, Network or Gateway Load Balancer target groups**, select its check box and choose a target group\.

   1. For **Classic Load Balancers**, select its check box and choose your load balancer\.

1. Choose **Update**\.

## Detach a load balancer<a name="as-remove-load-balancer"></a>

When you no longer need the load balancer, use the following procedure to detach it from your Auto Scaling group\.

**To detach a load balancer from a group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\.

1. On the **Details** tab, choose **Load balancing**, **Edit**\.

1. Under **Load balancing**, do one of the following:

   1. For **Application, Network or Gateway Load Balancer target groups**, choose the delete \(X\) icon next to the target group\.

   1. For **Classic Load Balancers**, choose the delete \(X\) icon next to the load balancer\. 

1. Choose **Update**\.