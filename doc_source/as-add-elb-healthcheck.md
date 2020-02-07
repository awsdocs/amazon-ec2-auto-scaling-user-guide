# Adding Elastic Load Balancing Health Checks to an Auto Scaling Group<a name="as-add-elb-healthcheck"></a>

The default health checks for an Auto Scaling group are EC2 status checks only\. If an instance fails these status checks, the Auto Scaling group considers the instance unhealthy and replaces it\. For more information, see [Health Checks for Auto Scaling Instances](healthcheck.md)\. 

If you attached one or more load balancers or target groups to your Auto Scaling group, by default the group does not consider an instance unhealthy and replace it if it fails the load balancer health checks\. 

However, you can optionally configure the Auto Scaling group to use Elastic Load Balancing health checks\. This ensures that the group can determine an instance's health based on additional tests provided by the load balancer\. The load balancer periodically sends pings, attempts connections, or sends requests to test the EC2 instances\. These tests are called health checks\. 

To learn more about Elastic Load Balancing health checks, see the following topics:
+ [Configure Health Checks for Your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-healthchecks.html) in the *User Guide for Classic Load Balancers*
+ [Health Checks for Your Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html) in the *User Guide for Application Load Balancers*
+ [Health Checks for Your Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html) in the *User Guide for Network Load Balancers*

If you configure the Auto Scaling group to use Elastic Load Balancing health checks, it considers the instance unhealthy if it fails either the EC2 status checks or the load balancer health checks\. If you attach multiple load balancers to an Auto Scaling group, all of them must report that the instance is healthy in order for it to consider the instance healthy\. If one load balancer reports an instance as unhealthy, the Auto Scaling group replaces the instance, even if other load balancers report it as healthy\. 

The following procedures show how to add Elastic Load Balancing health checks to your Auto Scaling group\.

**Topics**
+ [Adding Health Checks \(Console\)](#as-add-elb-healthcheck-console)
+ [Adding Health Checks \(AWS CLI\)](#as-add-elb-healthcheck-aws-cli)

## Adding Health Checks \(Console\)<a name="as-add-elb-healthcheck-console"></a>

Use the following procedure to add an Elastic Load Balancing \(ELB\) health check with a grace period of 300 seconds to an Auto Scaling group with an attached load balancer\.

**To add health checks**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Auto Scaling Groups**\.

1. Select the check box next to an existing group\.

   A new pane appears below the **Auto Scaling groups** pane, showing information about the group you selected\.

1. On the **Details** tab, under **Health checks**, choose **Edit**\. \(Old console: On the **Details** tab, choose **Edit**\.\)

1. For **Health check type**, select **Enable ELB health checks**\. \(Old console: Select **ELB**\.\)

1. For **Health check grace period**, enter `300`\.

1. Choose **Update**\. \(Old console: Choose **Save**\.\)

1. On the **Instance management** tab, under **Instances**, you can view the health status of instances\. \(Old console: The **Instances** tab is where you can view the health status of instances\.\) The **Health Status** column displays the results of the newly added health checks\.

## Adding Health Checks \(AWS CLI\)<a name="as-add-elb-healthcheck-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to create a health check with a grace period of 300 seconds\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-lb-asg \
  --health-check-type ELB --health-check-grace-period 300
```