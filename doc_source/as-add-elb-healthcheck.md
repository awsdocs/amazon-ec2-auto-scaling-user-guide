# Add Elastic Load Balancing health checks to an Auto Scaling group<a name="as-add-elb-healthcheck"></a>

The default health checks for an Auto Scaling group are EC2 health checks only\. If an instance fails these health checks, it is marked unhealthy and is terminated while Amazon EC2 Auto Scaling launches a new replacement instance\. For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\.

You can attach one or more load balancer target groups, one or more Classic Load Balancers, or both to your Auto Scaling group\. However, by default, the Auto Scaling group does not consider an instance unhealthy and replace it if it fails the Elastic Load Balancing health checks\. 

To ensure that your Auto Scaling group can determine instance health based on additional load balancer tests, configure the Auto Scaling group to use Elastic Load Balancing \(`ELB`\) health checks\. The load balancer periodically sends pings, attempts connections, or sends requests to test the EC2 instances and determines if an instance is unhealthy\. If you configure the Auto Scaling group to use Elastic Load Balancing health checks, it considers the instance unhealthy if it fails either the EC2 health checks or the Elastic Load Balancing health checks\. If you attach multiple load balancer target groups or Classic Load Balancers to the group, all of them must report that an instance is healthy in order for it to consider the instance healthy\. If any one of them reports an instance as unhealthy, the Auto Scaling group replaces the instance, even if others report it as healthy\. 

## Add Elastic Load Balancing health checks<a name="as-add-elb-healthcheck-console"></a>

To add Elastic Load Balancing health checks using the Amazon EC2 Auto Scaling console, perform the following steps\.

**To add Elastic Load Balancing health checks to a new group**  
When you create the Auto Scaling group, on the **Configure advanced options** page, for **Health checks**, **Additional health check types**, select **Turn on Elastic Load Balancing health checks**\. Then, for **Health check grace period**, enter the amount of time, in seconds\. This is how long Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance after it enters the `InService` state\. For more information, see [Set the health check grace period for an Auto Scaling group](health-check-grace-period.md)\.

**To add Elastic Load Balancing health checks to an existing group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. On the navigation bar at the top of the screen, choose the AWS Region that you created your Auto Scaling group in\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. For **Health checks**, **Additional health check types**, select **Turn on Elastic Load Balancing health checks**\.

1. For **Health check grace period**, enter the amount of time, in seconds\. This is how long Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance after it enters the `InService` state\. For more information, see [Set the health check grace period for an Auto Scaling group](health-check-grace-period.md)\. 

1. Choose **Update**\.

1. On the **Instance management** tab, under **Instances**, you can view the health status of instances\. The **Health Status** column displays the results of the newly added health checks\.

## See also<a name="elb-healthchecks-see-also"></a>
+ To configure health checks for your Application Load Balancer, see [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html) in the *User Guide for Application Load Balancers*\.
+ To configure health checks for your Network Load Balancer, see [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html) in the *User Guide for Network Load Balancers*\.
+ To configure health checks for your Gateway Load Balancer, see [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/health-checks.html) in the *User Guide for Gateway Load Balancers*\.
+ To configure health checks for your Classic Load Balancer, see [Configure health checks for your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-healthchecks.html) in the *User Guide for Classic Load Balancers*\.