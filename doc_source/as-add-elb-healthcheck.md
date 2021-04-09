# Adding Elastic Load Balancing health checks to an Auto Scaling group<a name="as-add-elb-healthcheck"></a>

The default health checks for an Auto Scaling group are EC2 status checks only\. If an instance fails these status checks, it is marked unhealthy and will be terminated while Amazon EC2 Auto Scaling launches a new instance in replacement\. 

You can attach one or more load balancer target groups, one or more Classic Load Balancers, or both to your Auto Scaling group\. However, by default, the Auto Scaling group does not consider an instance unhealthy and replace it if it fails the health checks provided by Elastic Load Balancing\. 

To ensure that your Auto Scaling group can determine an instance's health based on additional tests provided by the load balancer, you can configure the Auto Scaling group to use Elastic Load Balancing \(`ELB`\) health checks\. The load balancer periodically sends pings, attempts connections, or sends requests to test the EC2 instances and determines if an instance is unhealthy\. If you configure the Auto Scaling group to use `ELB` health checks, it considers the instance unhealthy if it fails either the EC2 status checks or the `ELB` health checks\. If you attach multiple load balancer target groups or Classic Load Balancers to the group, all of them must report that the instance is healthy in order for it to consider the instance healthy\. If any one of them reports an instance as unhealthy, the Auto Scaling group replaces the instance, even if other ones report it as healthy\. 

See the following topics:
+ To configure health checks for your Application Load Balancer, see [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html) in the *User Guide for Application Load Balancers*\.
+ To configure health checks for your Network Load Balancer, see [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html) in the *User Guide for Network Load Balancers*\.
+ To configure health checks for your Gateway Load Balancer, see [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/health-checks.html) in the *User Guide for Gateway Load Balancers*\.
+ To configure health checks for your Classic Load Balancer, see [Configure health checks for your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-healthchecks.html) in the *User Guide for Classic Load Balancers*\.
+ For more information about Amazon EC2 Auto Scaling health checks, see [Health checks for Auto Scaling instances](healthcheck.md)\. 

The following procedures show how to add `ELB` health checks to your Auto Scaling group\.

**Topics**
+ [Adding health checks \(console\)](#as-add-elb-healthcheck-console)
+ [Adding health checks \(AWS CLI\)](#as-add-elb-healthcheck-aws-cli)

## Adding health checks \(console\)<a name="as-add-elb-healthcheck-console"></a>

Use the following procedure to add an `ELB` health check to an Auto Scaling group with an attached load balancer\.

**To add health checks**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected\. 

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. For **Health check type**, select **Enable ELB health checks**\.

1. For **Health check grace period**, enter the amount of time, in seconds, that Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance\. Frequently, new instances need to briefly warm up before they can pass a health check\. To provide ample warm\-up time, set the health check grace period of the group to match the expected startup time of your application\.

1. Choose **Update**\.

1. On the **Instance management** tab, under **Instances**, you can view the health status of instances\. The **Health Status** column displays the results of the newly added health checks\.

## Adding health checks \(AWS CLI\)<a name="as-add-elb-healthcheck-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to create a health check with a grace period of 300 seconds\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-lb-asg \
  --health-check-type ELB --health-check-grace-period 300
```