# Using ELB Health Checks with Auto Scaling<a name="as-add-elb-healthcheck"></a>

An Auto Scaling group periodically checks the health status of each instance\. It can use EC2 status checks only, or EC2 status checks plus Elastic Load Balancing health checks\. If it determines that an instance is unhealthy, it replaces the instance\.

If you configure an Auto Scaling group to determine health status using EC2 status checks only, which is the default, It considers the instance unhealthy if it fails the EC2 status checks\. However, if you have attached one or more load balancers or target groups to the Auto Scaling group and a load balancer reports that an instance is unhealthy, it does not consider the instance unhealthy and therefore it does not replace it\.

If you configure your Auto Scaling group to determine health status using both EC2 status checks and Elastic Load Balancing health checks, it considers the instance unhealthy if it fails either the status checks or the health check\. Note that if you attach multiple load balancers to an Auto Scaling group, all of them must report that the instance is healthy in order for it to consider the instance healthy\. If one load balancer reports an instance as unhealthy, the Auto Scaling group replaces the instance, even if other load balancers report it as healthy\.

For more information, see [Health Checks for Auto Scaling Instances](healthcheck.md)\.


+ [Adding Health Checks Using the Console](#as-add-elb-healthcheck-console)
+ [Adding Health Checks Using the AWS CLI](#as-add-elb-healthcheck-aws-cli)

## Adding Health Checks Using the Console<a name="as-add-elb-healthcheck-console"></a>

Use the following procedure to add an `ELB` health check with a grace period of 300 seconds to an Auto Scaling group with an attached load balancer\.

**To add health checks using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your group\.

1. On the **Details** tab, choose **Edit**\.

1. For **Health Check Type**, select `ELB`\.

1. For **Health Check Grace Period**, enter `300`\.

1. Choose **Save**\.

1. On the **Instances** tab, the **Health Status** column displays the results of the newly added health checks\.

## Adding Health Checks Using the AWS CLI<a name="as-add-elb-healthcheck-aws-cli"></a>

Use the following [update\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to create a health check with a grace period of 300 seconds:

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-lb-asg --health-check-type ELB --health-check-grace-period 300
```