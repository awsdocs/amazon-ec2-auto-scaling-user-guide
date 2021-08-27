# Tutorial: Configure a lifecycle hook that invokes a Lambda function<a name="tutorial-lifecycle-hook-lambda"></a>

In this exercise, you create an EventBridge rule that includes a filter pattern that when matched, invokes an AWS Lambda function as the rule target\. We provide the filter pattern and sample function code to use\. 

If everything is configured correctly, at the end of this tutorial, the Lambda function performs a custom action when instances launch\. The custom action simply logs the event in the CloudWatch Logs log stream associated with the Lambda function\.

The Lambda function also performs a callback to let the lifecycle of the instance proceed if this action is successful, but lets the instance abandon the launch and terminate if the action fails\.

**Topics**
+ [Prerequisites](#lambda-hello-world-tutorial-prerequisites)
+ [Step 1: Create an IAM role with permissions to complete lifecycle hooks](#lambda-create-iam-role)
+ [Step 2: Create a Lambda function](#lambda-create-hello-world-function)
+ [Step 3: Create an EventBridge rule](#lambda-create-rule)
+ [Step 4: Add a lifecycle hook](#lambda-add-lifecycle-hook)
+ [Step 5: Test and verify the event](#lambda-testing-hook-notifications)
+ [Step 6: Next steps](#lambda-lifecycle-hooks-tutorial-next-steps)
+ [Step 7: Clean up](#lambda-lifecycle-hooks-tutorial-cleanup)

## Prerequisites<a name="lambda-hello-world-tutorial-prerequisites"></a>

Before you begin this tutorial, create an Auto Scaling group, if you don't have one already\. To create an Auto Scaling group, open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2autoscaling) in the Amazon EC2 console and choose **Create Auto Scaling group**\.

Note that all of the following procedures are for the new console\.

## Step 1: Create an IAM role with permissions to complete lifecycle hooks<a name="lambda-create-iam-role"></a>

Before you create a Lambda function, you must first create an execution role and a permissions policy to allow Lambda to complete lifecycle hooks\.

**To create the policy**

1. Open the [Policies page](https://console.aws.amazon.com/iam/home?#/policies) of the IAM console, and then choose **Create policy**\.

1. Choose the **JSON** tab\.

1. In the **Policy Document** box, paste the following policy document into the box, replacing the text in **italics** with your account number and the name of your Auto Scaling group\.

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "autoscaling:CompleteLifecycleAction"
         ],
         "Resource": "arn:aws:autoscaling:*:123456789012:autoScalingGroup:*:autoScalingGroupName/my-asg"
       }
     ]
   }
   ```

1. Choose **Next:Tags**, and then **Next:Review**\. 

1. For **Name**, enter **LogAutoScalingEvent\-policy**\. Choose **Create policy**\.

When you finish creating the policy, you can create a role that uses it\.

**To create the role**

1. On the navigation pane, choose **Roles**, **Create role**\.

1. Under **Choose a use case**, choose **Lambda** from the list, and then choose **Next:Permissions**\. 

1. Under **Attach permissions policies**, choose **LogAutoScalingEvent\-policy** and **AWSLambdaBasicExecutionRole**\. 

1. Choose **Next:Tags**, and then **Next:Review**\. 

1. On the **Review** page, for **Name**, enter **LogAutoScalingEvent\-role** and choose **Create role**\. 

## Step 2: Create a Lambda function<a name="lambda-create-hello-world-function"></a>

Create a Lambda function to serve as the target for events\. The sample Lambda function, written in Node\.js, is invoked by EventBridge when a matching event is emitted by Amazon EC2 Auto Scaling\.

**To create a Lambda function**

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) on the Lambda console\.

1. Choose **Create function**, **Author from scratch**\.

1. Under **Basic information**, for **Function name**, enter **LogAutoScalingEvent**\.

1. Choose **Change default execution role**, and then for **Execution role**, choose **Use an existing role**\.

1. For **Existing role**, choose **LogAutoScalingEvent\-role**\.

1. Leave the other default values\.

1. Choose **Create function**\. You are returned to the function's code and configuration\. 

1. With your `LogAutoScalingEvent` function still open in the console, under **Function code**, in the editor, copy the following sample code into the file named index\.js\.

   ```
   var aws = require("aws-sdk");
   exports.handler = (event, context, callback) => {
       console.log('LogAutoScalingEvent');
       console.log('Received event:', JSON.stringify(event, null, 2));
       var autoscaling = new aws.AutoScaling({region: event.region});
       var eventDetail = event.detail;
       var params = {
           AutoScalingGroupName: eventDetail['AutoScalingGroupName'], /* required */
           LifecycleActionResult: 'CONTINUE', /* required */
           LifecycleHookName: eventDetail['LifecycleHookName'], /* required */
           InstanceId: eventDetail['EC2InstanceId'],
           LifecycleActionToken: eventDetail['LifecycleActionToken']
       };
       var response; 
       autoscaling.completeLifecycleAction(params, function(err, data) {
           if (err) { 
               console.log(err, err.stack); // an error occurred
               response = {
                   statusCode: 500,
                   body: JSON.stringify('ERROR'),
               };
           } else {
               console.log(data);           // successful response
               response = {
                   statusCode: 200,
                   body: JSON.stringify('SUCCESS'),
               };
           }
       });
       return response;
   };
   ```

   This code simply logs the event so that, at the end of this tutorial, you can see an event appear in the CloudWatch Logs log stream that's associated with this Lambda function\. 

1. Choose **Deploy**\. 

## Step 3: Create an EventBridge rule<a name="lambda-create-rule"></a>

Create an EventBridge rule to run your Lambda function\.

**To create a rule using the console**

1. Open the [EventBridge console](https://console.aws.amazon.com/events/)\.

1. On the navigation pane, choose **Rules**, **Create rule**\.

1. For **Name**, enter **LogAutoScalingEvent\-rule**\.

1. For **Define pattern**, choose **Event Pattern**\.

1. For **Event matching pattern**, choose **Custom pattern**\.

1. Rules use event patterns to select events and route them to targets\. Copy the following pattern into the **Event pattern** box\.

   ```
   {
     "source": [ "aws.autoscaling" ],
     "detail-type": [ "EC2 Instance-launch Lifecycle Action" ]
   }
   ```

1. To save the event pattern, choose **Save**\. 

1. For **Select event bus**, choose **AWS default event bus**\. 

1. For **Target**, choose **Lambda function**\.

1. For **Function**, select **LogAutoScalingEvent**\. Choose **Create**\.

## Step 4: Add a lifecycle hook<a name="lambda-add-lifecycle-hook"></a>

In this section, you add a lifecycle hook so that Lambda runs your function on instances at launch\.

**To add a lifecycle hook**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2autoscaling) in the Amazon EC2 console\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page\. 

1. In the lower pane, on the **Instance management** tab, in **Lifecycle hooks**, choose **Create lifecycle hook**\.

1. To define a lifecycle hook, do the following:

   1. For **Lifecycle hook name**, enter **LogAutoScalingEvent\-hook**\.

   1. For **Lifecycle transition**, choose **Instance launch**\.

   1. For **Heartbeat timeout**, enter **300** for the number of seconds to wait for a callback from your Lambda function\.

   1. For **Default result**, choose **ABANDON**\. This means that the Auto Scaling group will terminate a new instance if the hook times out without receiving a callback from your Lambda function\.

   1. \(Optional\) Leave **Notification metadata** empty\. The event data that we pass to EventBridge contains all of the necessary information to invoke the Lambda function\.

1. Choose **Create**\.

## Step 5: Test and verify the event<a name="lambda-testing-hook-notifications"></a>

To test the event, update the Auto Scaling group by increasing the desired capacity of the Auto Scaling group by 1\. Your Lambda function is invoked within a few seconds after increasing the desired capacity\.

**To increase the size of the Auto Scaling group**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2autoscaling) in the Amazon EC2 console\.

1. Select the check box next to your Auto Scaling group to view details in a lower pane and still see the top rows of the upper pane\. 

1. In the lower pane, on the **Details** tab, choose **Group details**, **Edit**\.

1. For **Desired capacity**, increase the current value by 1\.

1. Choose **Update**\. While the instance is being launched, the **Status** column in the upper pane displays a status of *Updating capacity*\. 

After increasing the desired capacity, you can verify that your Lambda function was invoked\.

**To view the output from your Lambda function**

1. Open the [Log groups page](https://console.aws.amazon.com/cloudwatch/home#logs:) of the CloudWatch console\.

1. Select the name of the log group for your Lambda function \(`/aws/lambda/LogAutoScalingEvent`\)\.

1. Select the name of the log stream to view the data provided by the function for the lifecycle action\.

Next, you can verify that your instance has successfully launched from the description of scaling activities\.

**To view the scaling activity**

1. Return to the **Auto Scaling groups** page and select your group\.

1. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched an instance\. 
   + If the action was successful, the scaling activity will have a status of "Successful"\.
   + If it failed, after waiting a few minutes, you will see a scaling activity with a status of "Cancelled" and a status message of "Instance failed to complete user's Lifecycle Action: Lifecycle Action with token e85eb647\-4fe0\-4909\-b341\-a6c42EXAMPLE was abandoned: Lifecycle Action Completed with ABANDON Result"\.

**To decrease the size of the Auto Scaling group**  
If you do not need the additional instance that you launched for this test, you can open the **Details** tab and decrease **Desired capacity** by 1\.

## Step 6: Next steps<a name="lambda-lifecycle-hooks-tutorial-next-steps"></a>

Now that you have completed this tutorial, you can try creating a termination lifecycle hook\. If instances in the Auto Scaling group terminate, an event is sent to EventBridge\. For information about the event that is emitted when an instance terminates, see [EC2 Instance\-terminate Lifecycle Action](cloud-watch-events.md#terminate-lifecycle-action)\.

## Step 7: Clean up<a name="lambda-lifecycle-hooks-tutorial-cleanup"></a>

If you are done working with the resources that you created just for this tutorial, use the following steps to delete them\.

**To delete the lifecycle hook**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2autoscaling) in the Amazon EC2 console\.

1. Select the check box next to your Auto Scaling group\.

1. On the **Instance management** tab, in **Lifecycle hooks**, choose the lifecycle hook \(`LogAutoScalingEvent-hook`\)\.

1. Choose **Actions**, **Delete**\.

1. Choose **Delete** again to confirm\.

**To delete the Amazon EventBridge rule**

1. Open the [Rules page](https://console.aws.amazon.com/events/home?#/rules) in the Amazon EventBridge console\.

1. Under **Event bus**, choose the event bus that is associated with the rule \(`Default`\)\.

1. Choose the rule \(`LogAutoScalingEvent-rule`\)\.

1. Choose **Actions**, **Delete**\.

1. Choose **Delete** again to confirm\.

If you are done working with the example function, delete it\. You can also delete the log group that stores the function's logs, and the execution role and permissions policy that you created\.

**To delete a Lambda function**

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) on the Lambda console\.

1. Choose the function \(`LogAutoScalingEvent`\)\.

1. Choose **Actions**, **Delete**\.

1. In the **Delete function** dialog box, choose **Delete**\.

**To delete the log group**

1. Open the [Log groups page](https://console.aws.amazon.com/cloudwatch/home#logs:) of the CloudWatch console\.

1. Select the function's log group \(`/aws/lambda/LogAutoScalingEvent`\)\.

1. Choose **Actions**, **Delete log group\(s\)**\.

1. In the **Delete log group\(s\)** dialog box, choose **Delete**\.

**To delete the execution role**

1. Open the [Roles page](https://console.aws.amazon.com/iam/home?#/roles) of the IAM console\.

1. Select the function's role \(`LogAutoScalingEvent-role`\)\.

1. Choose **Delete role**\.

1. In the **Delete role** dialog box, choose **Yes, delete**\.

**To delete the IAM policy**

1. Open the [Policies page](https://console.aws.amazon.com/iam/home?#/policies) of the IAM console\.

1. Select the policy that you created \(`LogAutoScalingEvent-policy`\)\.

1. Choose **Delete policy**\.

1. In the **Delete policy** dialog box, choose **Yes, delete**\.