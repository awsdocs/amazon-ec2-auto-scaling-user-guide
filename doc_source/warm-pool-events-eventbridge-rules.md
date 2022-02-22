# Creating EventBridge rules for warm pool events<a name="warm-pool-events-eventbridge-rules"></a>

The following procedures explain how to create an EventBridge rule for warm pool events\. This sample rule detects events that use the event pattern for instances entering the warm pool, and then sends those events to an AWS Lambda function for processing\. A Lambda target is the subject of this procedure, but rules can invoke many types of targets\. For information about supported targets, see [Amazon EventBridge targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-targets.html) in the *Amazon EventBridge User Guide*\.

Before you create the rule, create the AWS Lambda function that you want the rule to use as a target\. When you create the rule, you'll need to specify this function as the target for the rule\. For an introductory tutorial that shows you how to create a Lambda function, see [Tutorial: Configure a lifecycle hook that invokes a Lambda function](tutorial-lifecycle-hook-lambda.md)\.

## Create an EventBridge rule \(console\)<a name="warm-pool-events-eventbridge-rules-console"></a>

**To create an event rule**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, under **Events**, choose **Rules**\.

1. In the **Rules** section, choose **Create rule**\.

1. For **Name**, enter a name for the rule\. Optionally enter a description of the rule in the **Description** box\.

1. Under **Define pattern**, choose **Event pattern**\.

1. For **Event matching pattern**, choose **Custom pattern**\.

1. Rules use event patterns to select events and route them to targets\. Copy the following pattern into the **Event pattern** box\.

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-launch Lifecycle Action" ],
     "detail": {
         "Origin": [ "EC2" ],
         "Destination": [ "WarmPool" ]
      }
   }
   ```

1. To save the event pattern, choose **Save**\. 

1. For **Select event bus**, choose **AWS default event bus**\.

1. Under **Select targets**, for **Target**, choose **Lambda function**\. Then, for **Function**, choose the function that you want to send the events to\. 

1. When you finish entering settings for the rule, choose **Create**\.

1. After you have followed these instructions, continue on to [Adding lifecycle hooks](adding-lifecycle-hooks.md) as a next step\.

## Create an EventBridge rule \(AWS CLI\)<a name="warm-pool-events-eventbridge-rules-cli"></a>

If you haven't already, create the Lambda function that you want the rule to use as a target\. When you create the function, note the Amazon Resource Name \(ARN\) of the function\. You'll need to enter this ARN when you specify the target for the rule\.

**To create an event rule**

1. To create a rule that matches events for instances entering the warm pool, use the following [put\-rule](https://docs.aws.amazon.com/cli/latest/reference/events/put-rule.html) command\.

   ```
   aws events put-rule --name my-rule --event-pattern file://pattern.json --state ENABLED
   ```

   The following example shows the `pattern.json` to match the event for instances entering the warm pool\.

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-launch Lifecycle Action" ],
     "detail": {
         "Origin": [ "EC2" ],
         "Destination": [ "WarmPool" ]
      }
   }
   ```

   If the command runs successfully, EventBridge responds with the ARN of the rule\. Note this ARN\. You'll need to enter it in step 3\.

1. To specify the Lambda function to use as a target for the rule, use the following [put\-targets](https://docs.aws.amazon.com/cli/latest/reference/events/put-targets.html) command\.

   ```
   aws events put-targets --rule my-rule --targets Id=1,Arn=arn:aws:lambda:region:123456789012:function:my-function
   ```

   In the preceding command, *my\-rule* is the name that you specified for the rule in step 1, and the value for the `Arn` parameter is the ARN of the function that you want the rule to use as a target\.

1. To add permissions that allow the rule to invoke the target Lambda function, use the following Lambda [add\-permission](https://docs.aws.amazon.com/cli/latest/reference/lambda/add-permission.html) command\.

   ```
   aws lambda add-permission --function-name my-function --statement-id AllowInvokeFromExampleEvent \
     --action 'lambda:InvokeFunction' --principal events.amazonaws.com --source-arn arn:aws:events:region:123456789012:rule/my-rule
   ```

   In the preceding command:
   + *my\-function* is the name of the Lambda function that you want the rule to use as a target\.
   + *AllowInvokeFromExampleEvent* is a unique identifier that you define to describe the statement in the Lambda function policy\.
   + `source-arn` is the ARN of the EventBridge rule\.

   If the command runs successfully, you receive output similar to the following\.

   ```
   {
     "Statement": "{\"Sid\":\"AllowInvokeFromExampleEvent\",
       \"Effect\":\"Allow\",
       \"Principal\":{\"Service\":\"events.amazonaws.com\"},
       \"Action\":\"lambda:InvokeFunction\",
       \"Resource\":\"arn:aws:lambda:us-west-2:123456789012:function:my-function\",
       \"Condition\":
         {\"ArnLike\":
           {\"AWS:SourceArn\":
            \"arn:aws:events:us-west-2:123456789012:rule/my-rule\"}}}"
   }
   ```

   The `Statement` value is a JSON string version of the statement that was added to the Lambda function policy\.

1. After you have followed these instructions, continue on to [Adding lifecycle hooks](adding-lifecycle-hooks.md) as a next step\.