# Create EventBridge rules for warm pool events<a name="warm-pool-events-eventbridge-rules"></a>

The following example creates an EventBridge rule to invoke programmatic actions\. It does this each time that your Auto Scaling group emits an event when a new instance is added to the warm pool\.

Before you create the rule, create the AWS Lambda function that you want the rule to use as a target\. You must specify this function as the target for the rule\. The following procedure provides only the steps for creating the EventBridge rule that acts when new instances enter the warm pool\. For an introductory tutorial that shows you how to create a simple Lambda function to invoke when an incoming event matches a rule, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\.

For more information about creating and working with warm pools, see [Warm pools for Amazon EC2 Auto Scaling](ec2-auto-scaling-warm-pools.md)\.

**To create an event rule that invokes a Lambda function**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, choose **Rules**\.

1. Choose **Create rule**\.

1. For **Define rule detail**, do the following:

   1. Enter a **Name** for the rule, and, optionally, a description\.

      A rule can't have the same name as another rule in the same Region and on the same event bus\.

   1. For **Event bus**, choose **default**\. When an AWS service in your account generates an event, it always goes to your account's default event bus\.

   1. For **Rule type**, choose **Rule with an event pattern**\.

   1. Choose **Next**\.

1. For **Build event pattern**, do the following:

   1. For **Event source**, choose **AWS events or EventBridge partner events**\.

   1. For **Event pattern**, choose **Custom pattern \(JSON editor\)**, and paste the following pattern into the **Event pattern** box, replacing the text in **italics** with the name of your Auto Scaling group\.

      ```
      {
        "source": [ "aws.autoscaling" ],
        "detail-type": [ "EC2 Instance-launch Lifecycle Action" ],
        "detail": {
            "AutoScalingGroupName": [ "my-asg" ],
            "Origin": [ "EC2" ],
            "Destination": [ "WarmPool" ]
         }
      }
      ```

      To create a rule that matches for other events, modify the event pattern\. For more information, see [Example event patterns](warm-pools-eventbridge-events.md#warm-pools-eventbridge-patterns)\.

   1. Choose **Next**\.

1. For **Select target\(s\)**, do the following: 

   1. For **Target types**, choose **AWS service**\.

   1. For **Select a target**, choose **Lambda function**\.

   1. For **Function**, choose the function that you want to send the events to\.

   1. \(Optional\) For **Configure version/alias**, enter version and alias settings for the target Lambda function\.

   1. \(Optional\) For **Additional settings**, enter any additional settings as appropriate for your application\. For more information, see [Creating Amazon EventBridge rules that react to events](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule.html) \(step 16\) in the *Amazon EventBridge User Guide*\.

   1. Choose **Next**\.

1. \(Optional\) For **Tags**, you can optionally assign one or more tags to your rule, and then choose **Next**\.

1. For **Review and create**, review the details of the rule and modify them as necessary\. Then, choose **Create rule**\.