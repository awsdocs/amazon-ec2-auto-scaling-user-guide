# Create a custom termination policy with Lambda<a name="lambda-custom-termination-policy"></a>

Amazon EC2 Auto Scaling uses termination policies to prioritize which instances to terminate first when decreasing the size of your Auto Scaling group \(referred to as *scaling in*\)\. Your Auto Scaling group uses a default termination policy, but you can optionally choose or create your own termination policies\. For more information about choosing a predefined termination policy, see [Work with Amazon EC2 Auto Scaling termination policies](ec2-auto-scaling-termination-policies.md)\.

In this topic, you learn how to create a custom termination policy using an AWS Lambda function that Amazon EC2 Auto Scaling invokes in response to certain events\. The Lambda function that you create processes the information in the input data sent by Amazon EC2 Auto Scaling and returns a list of instances that are ready to terminate\.

A custom termination policy provides better control over which instances are terminated, and when\. For example, when your Auto Scaling group scales in, Amazon EC2 Auto Scaling cannot determine whether there are workloads running that should not be disrupted\. With a Lambda function, you can validate the termination request and wait until the workload is done before returning the instance ID to Amazon EC2 Auto Scaling for termination\. 

**Topics**
+ [Input data](#lambda-custom-termination-policy-input-data)
+ [Response data](#lambda-custom-termination-policy-response-data)
+ [Considerations](#lambda-termination-policy-considerations)
+ [Create the Lambda function](#lambda-custom-termination-policy-create-function)
+ [Limitations](#lambda-custom-termination-policy-limitations)

## Input data<a name="lambda-custom-termination-policy-input-data"></a>

Amazon EC2 Auto Scaling generates a JSON payload for scale\-in events, and also does so when instances are about to be terminated as a result of the maximum instance lifetime or instance refresh features\. It also generates a JSON payload for the scale\-in events that it can initiate when rebalancing your group across Availability Zones\.

This payload contains information about the capacity Amazon EC2 Auto Scaling needs to terminate, a list of instances that it suggests for termination, and the event that initiated the termination\. 

The following is an example payload:

```
{
  "AutoScalingGroupARN": "arn:aws:autoscaling:us-east-1:<account-id>:autoScalingGroup:d4738357-2d40-4038-ae7e-b00ae0227003:autoScalingGroupName/my-asg",
  "AutoScalingGroupName": "my-asg",
  "CapacityToTerminate": [
    {
      "AvailabilityZone": "us-east-1b",
      "Capacity": 2,
      "InstanceMarketOption": "on-demand"
    },
    {
      "AvailabilityZone": "us-east-1b",
      "Capacity": 1,
      "InstanceMarketOption": "spot"
    },
    {
      "AvailabilityZone": "us-east-1c",
      "Capacity": 3,
      "InstanceMarketOption": "on-demand"
    }
  ],
  "Instances": [
    {
      "AvailabilityZone": "us-east-1b",
      "InstanceId": "i-0056faf8da3e1f75d",
      "InstanceType": "t2.nano",
      "InstanceMarketOption": "on-demand"
    },
    {
      "AvailabilityZone": "us-east-1c",
      "InstanceId": "i-02e1c69383a3ed501",
      "InstanceType": "t2.nano",
      "InstanceMarketOption": "on-demand"
    },
    {
      "AvailabilityZone": "us-east-1c",
      "InstanceId": "i-036bc44b6092c01c7",
      "InstanceType": "t2.nano",
      "InstanceMarketOption": "on-demand"
    },
    ...
  ],
  "Cause": "SCALE_IN"
}
```

The payload includes the name of the Auto Scaling group, its Amazon Resource Name \(ARN\), and the following elements:
+ `CapacityToTerminate` describes how much of your Spot or On\-Demand capacity is set to be terminated in a given Availability Zone\. 
+ `Instances` represents the instances that Amazon EC2 Auto Scaling suggests for termination based on the information in `CapacityToTerminate`\. 
+ `Cause` describes the event that caused the termination: `SCALE_IN`, `INSTANCE_REFRESH`, `MAX_INSTANCE_LIFETIME`, or `REBALANCE`\. 

The following information outlines the most significant factors in how Amazon EC2 Auto Scaling generates the `Instances` in the input data:
+ Maintaining balance across Availability Zones takes precedence when an instance is terminating due to scale\-in events and instance refresh\-based terminations\. Therefore, if one Availability Zone has more instances than the other Availability Zones that are used by the group, the input data contains instances that are eligible for termination only from the imbalanced Availability Zone\. If the Availability Zones used by the group are balanced, the input data contains instances from all of the Availability Zones for the group\. 
+ When using a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md), maintaining your Spot and On\-Demand capacities in balance based on your desired percentages for each purchase option also takes precedence\. We first identify which of the two types \(Spot or On\-Demand\) should be terminated\. We then identify which instances \(within the identified purchase option\) in which Availability Zones we can terminate that will result in the Availability Zones being most balanced\. 

## Response data<a name="lambda-custom-termination-policy-response-data"></a>

The input data and response data work together to narrow down the list of instances to terminate\. 

With the given input, the response from your Lambda function should look like the following example:

```
{
  "InstanceIDs": [
    "i-02e1c69383a3ed501",
    "i-036bc44b6092c01c7",
    ...
  ]
}
```

The `InstanceIDs` in the response represent the instances that are ready to terminate\. 

Alternatively, you can return a different set of instances that are ready to be terminated, which overrides the instances in the input data\. If no instances are ready to terminate when your Lambda function is invoked, you can also choose not to return any instances\.

When no instances are ready to terminate, the response from your Lambda function should look like the following example:

```
{
  "InstanceIDs": [ ]
}
```

## Considerations<a name="lambda-termination-policy-considerations"></a>

Note the following considerations when using a custom termination policy:
+ Returning an instance first in the response data does not guarantee its termination\. If more than the required number of instances are returned when your Lambda function is invoked, Amazon EC2 Auto Scaling evaluates each instance against the other termination policies that you specified for your Auto Scaling group\. When there are multiple termination policies, it tries to apply the next termination policy in the list, and if there are more instances than are required to terminate, it moves on to the next termination policy, and so on\. If no other termination policies are specified, then the default termination policy is used to determine which instances to terminate\.
+ If no instances are returned or your Lambda function times out, then Amazon EC2 Auto Scaling waits a short time before invoking your function again\. For any scale\-in event, it keeps trying as long as the group's desired capacity is less than its current capacity\. For instance refresh\-based terminations, it keeps trying for an hour\. After that, if it continues to fail to terminate any instances, the instance refresh operation fails\. With maximum instance lifetime, Amazon EC2 Auto Scaling keeps trying to terminate the instance that is identified as exceeding its maximum lifetime\. 
+ Because your function is retried repeatedly, make sure to test and fix any permanent errors in your code before using a Lambda function as a custom termination policy\.
+ If you override the input data with your own list of instances to terminate, and terminating these instances puts the Availability Zones out of balance, Amazon EC2 Auto Scaling gradually rebalances the distribution of capacity across Availability Zones\. First, it invokes your Lambda function to see if there are instances that are ready to be terminated so that it can determine whether to start rebalancing\. If there are instances ready to be terminated, it launches new instances first\. When the instances finish launching, it then detects that your group's current capacity is higher than its desired capacity and initiates a scale\-in event\.
+ A custom termination policy does not affect your ability to also use scale\-in protection to protect certain instances from being terminated\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

## Create the Lambda function<a name="lambda-custom-termination-policy-create-function"></a>

Start by creating the Lambda function, so that you can specify its Amazon Resource Name \(ARN\) in the termination policies for your Auto Scaling group\.

**To create a Lambda function \(console\)**

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) on the Lambda console\.

1. On the navigation bar at the top of the screen, choose the same Region that you used when you created the Auto Scaling group\. 

1. Choose **Create function**, **Author from scratch**\.

1. Under **Basic information**, for **Function name**, enter the name of your function\.

1. Choose **Create function**\. You are returned to the function's code and configuration\. 

1. With your function still open in the console, under **Function code**, paste your code into the editor\.

1. Choose **Deploy**\. 

1. Optionally, create a published version of the Lambda function by choosing the **Versions** tab and then **Publish new version**\. To learn more about versioning in Lambda, see [Lambda function versions](https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html) in the *AWS Lambda Developer Guide*\.

1. If you chose to publish a version, choose the **Aliases** tab if you want to associate an alias with this version of the Lambda function\. To learn more about aliases in Lambda, see [Lambda function aliases](https://docs.aws.amazon.com/lambda/latest/dg/configuration-aliases.html) in the *AWS Lambda Developer Guide*

1. Next, choose the **Configuration** tab and then **Permissions**\. 

1. Scroll down to **Resource\-based policy** and then choose **Add permissions**\. A resource\-based policy is used to grant permissions to invoke your function to the principal that is specified in the policy\. In this case, the principal will be the [Amazon EC2 Auto Scaling service\-linked role](https://docs.aws.amazon.com/autoscaling/ec2/userguide/autoscaling-service-linked-role.html) that is associated with the Auto Scaling group\.

1. In the **Policy statement** section, configure your permissions: 

   1. Choose **AWS account**\.

   1. For **Principal**, enter the ARN of the calling service\-linked role, for example, **arn:aws:iam::<aws\-account\-id>:role/aws\-service\-role/autoscaling\.amazonaws\.com/AWSServiceRoleForAutoScaling**\.

   1. For **Action**, choose **lambda:InvokeFunction**\.

   1. For **Statement ID**, enter a unique statement ID, such as **AllowInvokeByAutoScaling**\.

   1. Choose **Save**\. 

1. After you have followed these instructions, continue on to specify the ARN of your function in the termination policies for your Auto Scaling group as a next step\. For more information, see [Use different termination policies \(console\)](ec2-auto-scaling-termination-policies.md#custom-termination-policy-console)\. 

**Note**  
For examples that you can use as a reference for developing your Lambda function, see the [GitHub repository](https://github.com/aws-samples/amazon-ec2-auto-scaling-group-examples) for Amazon EC2 Auto Scaling\.

## Limitations<a name="lambda-custom-termination-policy-limitations"></a>
+ You can only specify one Lambda function in the termination policies for an Auto Scaling group\. If there are multiple termination policies specified, the Lambda function must be specified first\.
+ You can reference your Lambda function using either an unqualified ARN \(without a suffix\) or a qualified ARN that has either a version or an alias as its suffix\. If an unqualified ARN is used \(for example, `function:my-function`\), your resource\-based policy must be created on the unpublished version of your function\. If a qualified ARN is used \(for example, `function:my-function:1` or `function:my-function:prod`\), your resource\-based policy must be created on that specific published version of your function\.
+ You cannot use a qualified ARN with the `$LATEST` suffix\. If you try to add a custom termination policy that refers to a qualified ARN with the `$LATEST` suffix, it will result in an error\.
+ The number of instances provided in the input data is limited to 30,000 instances\. If there are more than 30,000 instances that could be terminated, the input data includes `"HasMoreInstances": true` to indicate that the maximum number of instances are returned\. 
+ The maximum run time for your Lambda function is two seconds \(2000 milliseconds\)\. As a best practice, you should set the timeout value of your Lambda function based on your expected run time\. Lambda functions have a default timeout of three seconds, but this can be decreased\.