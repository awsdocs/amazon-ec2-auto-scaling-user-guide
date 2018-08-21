# Add a Scaling Policy to an Existing Auto Scaling Group<a name="policy-updating-console"></a>

Use the console to add a scaling policy to an existing Auto Scaling group\. A scaling policy is used to increase and decrease the number of running EC2 instances in the group dynamically to meet changing conditions\. For more information, see [Dynamic Scaling for Amazon EC2 Auto Scaling](as-scale-based-on-demand.md)\.

**To update an Auto Scaling group with scaling based on metrics**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select the Auto Scaling group\.

1. On the **Scaling Policies** tab, choose **Add policy**\. 

   The page displays all the available resources\.

1. If you are adding a target tracking scaling policy, use the following steps\. If you are using a simple or step scaling policy, skip to the next step\.

   1. For **Name**, type a name for the policy\.

   1. Choose a **Metric type** and specify a **Target value** for the metric\.

   1. Specify an instance warm\-up value for **Instances need**, which allows you to control the amount of time until a newly launched instance can contribute to the CloudWatch metrics\.

   1. Check the **Disable scale\-in** option if you only want a scale\-out policy created\. This allows you to create a separate scale\-in policy if wanted\.

   1. Choose **Create**\.

1. If you are using a step scaling policy, choose **Create a scaling policy with steps**, and then do the following:
**Note**  
We recommend that you use the option to create scaling policies with steps\. To use a simple scaling policy, choose **Create a simple scaling policy** instead\. For more information about simple scaling, see [Simple and Step Scaling Policies for Amazon EC2 Auto Scaling](as-scaling-simple-step.md)\.

   1. For **Name**, type a name for the policy, and then choose **Create new alarm**\.

   1. On the **Create Alarm** page, choose **create topic**\. For **Send a notification to**, type a name for the SNS topic\. For **With these recipients**, type one or more email addresses to receive notification\. If you want, you can replace the default name for your alarm with a custom name\. Next, specify the metric and the criteria for the alarm, using **Whenever**, **Is**, and **For at least**\. Choose **Create Alarm**\.

   1. Specify the scaling activity for the policy using **Take the action**\. By default, the lower bound for this step adjustment is the alarm threshold and the upper bound is null \(positive infinity\)\. To add another step adjustment, choose **Add step**\.

   1. Specify an instance warm\-up value for **Instances need**, which allows you to control the amount of time until a newly launched instance can contribute to the CloudWatch metrics\. 

   1. Choose **Create**\.