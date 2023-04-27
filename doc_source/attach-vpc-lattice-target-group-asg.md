# Attach a VPC Lattice target group to your Auto Scaling group<a name="attach-vpc-lattice-target-group-asg"></a>

This topic describes how to attach a VPC Lattice target group to an Auto Scaling group\. It also describes how to turn on VPC Lattice health checks to let Amazon EC2 Auto Scaling replace instances that VPC Lattice reports as unhealthy\. 

By default, Amazon EC2 Auto Scaling only replaces instances that are unhealthy or unreachable based on Amazon EC2 health checks\. If you turn on VPC Lattice health checks, Amazon EC2 Auto Scaling can replace a running instance if any of the VPC Lattice target groups you attach to the Auto Scaling group report it as unhealthy\. For more information, see [Health checks for Auto Scaling instances](ec2-auto-scaling-health-checks.md)\.

**Important**  
Before you continue, complete all [prerequisites](getting-started-vpc-lattice.md) in the previous section\.

## Attach a VPC Lattice target group<a name="attach-vpc-lattice-target-group"></a>

You can attach one or multiple target groups to an Auto Scaling group when you create or update the group\.

------
#### [ Console ]

Follow the steps in this section to use the console to:
+ Attach a VPC Lattice target group to an Auto Scaling group
+ Turn on the health checks for VPC Lattice

**To attach a VPC Lattice target group to a new Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. On the navigation bar at the top of the screen, choose the AWS Region that you created your target group in\.

1. Choose **Create Auto Scaling group**\.

1.  In steps 1 and 2, choose your desired options and proceed to **Step 3: Configure advanced options**\.

1. For **VPC Lattice integration options**, choose **Attach to VPC Lattice service**\.

1. Under **Choose VPC Lattice target group**, choose your target group\.

1. \(Optional\) For **Health checks**, **Additional health check types**, select **Turn on VPC Lattice health checks**\.

1. \(Optional\) For **Health check grace period**, enter the amount of time, in seconds\. This is how long Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance after it enters the `InService` state\. For more information, see [Set the health check grace period for an Auto Scaling group](health-check-grace-period.md)\. 

1. Proceed to create the Auto Scaling group\. Your instances will be automatically registered to the VPC Lattice target group after the Auto Scaling group has been created\. 

**To attach a VPC Lattice target group to an existing Auto Scaling group**

Use the following procedure to attach a target group for a service to an existing Auto Scaling group\. 

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom of the page\. 

1. On the **Details** tab, choose **VPC Lattice integration options**, **Edit**\.

1. Under **VPC Lattice integration options**, choose **Attach to VPC Lattice service**\.

1. Under **Choose VPC Lattice target group**, choose your target group\.

1. Choose **Update**\.

When you finish attaching the target group, you can optionally turn on the health checks that use it\.

**To turn on the VPC Lattice health checks**

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. For **Health checks**, **Additional health check types**, select **Turn on VPC Lattice health checks**\.

1. For **Health check grace period**, enter the amount of time, in seconds\. This is how long Amazon EC2 Auto Scaling needs to wait before checking the health status of an instance after it enters the `InService` state\. For more information, see [Set the health check grace period for an Auto Scaling group](health-check-grace-period.md)\. 

1. Choose **Update**\.

------
#### [ AWS CLI ]

Follow the steps in this section to use the AWS CLI to:
+ Attach a VPC Lattice target group to an Auto Scaling group
+ Turn on the health checks for VPC Lattice

**To attach a VPC Lattice target group to an Auto Scaling group**

Use the following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group and simultaneously attach a VPC Lattice target group by specifying its Amazon Resource Name \(ARN\)\. 

Replace the sample values for `--auto-scaling-group-name`, `--vpc-zone-identifier`, `--min-size`, and `--max-size`\. For the `--launch-template` option, replace `my-launch-template` and `1` with the name and version of the launch template that you created for instances registered to a VPC Lattice target group\. For the `--traffic-sources` option, replace the sample ARN with the ARN of your VPC Lattice target group\. 

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-launch-template,Version='1' \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --min-size 1 --max-size 5 \
  --traffic-sources "Identifier=arn:aws:vpc-lattice:region:account-id:targetgroup/tg-0e2f2665eEXAMPLE"
```

Use the following [attach\-traffic\-sources](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/attach-traffic-sources.html) command to attach a VPC Lattice target group to an Auto Scaling group after it's already created\.

```
aws autoscaling attach-traffic-sources --auto-scaling-group-name my-asg \
  --traffic-sources "Identifier=arn:aws:vpc-lattice:region:account-id:targetgroup/tg-0e2f2665eEXAMPLE"
```

**To turn on the health checks for VPC Lattice**

If you have configured an application\-based health check for your **VPC Lattice** target group, you can turn on these health checks\. Use the [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) or [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command with the `--health-check-type` option and a value of `VPC_LATTICE`\. To specify the grace period for the health checks performed by your Auto Scaling group, include the `--health-check-grace-period` option and provide its value in seconds\.

```
--health-check-type "VPC_LATTICE" --health-check-grace-period 60
```

------

## Detach a VPC Lattice target group<a name="detach-vpc-lattice-target-group"></a>

If you no longer need to use VPC Lattice, use the following procedure to detach the target group from your Auto Scaling group\.

------
#### [ Console ]

Follow the steps in this section to use the console to:
+ Detach a VPC Lattice target group from an Auto Scaling group
+ Turn off the health checks for VPC Lattice

**To detach a VPC Lattice target group from an Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom of the page\.

1. On the **Details** tab, choose **VPC Lattice integration options**, **Edit**\.

1. Under **VPC Lattice integration options**, choose the delete \(X\) icon next to the target group\.

1. Choose **Update**\.

When you finish detaching the target group, you can turn off the VPC Lattice health checks\.

**To turn off the VPC Lattice health checks**

1. On the **Details** tab, choose **Health checks**, **Edit**\.

1. For **Health checks**, **Additional health check types**, deselect **Turn on VPC Lattice health checks**\.

1. Choose **Update**\.

------
#### [ AWS CLI ]

Follow the steps in this section to use the AWS CLI to:
+ Detach a VPC Lattice target group from an Auto Scaling group
+ Turn off the health checks for VPC Lattice

Use the [detach\-traffic\-sources](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-traffic-sources.html) command to detach a target group from your Auto Scaling group when you no longer need it\. 

```
aws autoscaling detach-traffic-sources --auto-scaling-group-name my-asg \
  --traffic-sources "Identifier=arn:aws:vpc-lattice:region:account-id:targetgroup/tg-0e2f2665eEXAMPLE"
```

To update the health checks on an Auto Scaling group so that it no longer uses VPC Lattice health checks, use the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command\. Include the `--health-check-type` option and a value of `EC2`\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --health-check-type "EC2"
```

------