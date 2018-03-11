# Tagging Auto Scaling Groups and Instances<a name="autoscaling-tagging"></a>

You can organize and manage your Auto Scaling groups by assigning your own metadata to each group in the form of *tags*\. You specify a *key* and a *value* for each tag\. A key can be a general category, such as "project", "owner", or "environment", with specific associated values\. For example, to differentiate between your testing and production environments, you could assign each Auto Scaling group a tag with a key of "environment", and either a value of "test" to indicate your test environment or "production" to indicate your production environment\. We recommend that you use a consistent set of tags to make it easier to track your Auto Scaling groups\.

You can specify that the tags for your Auto Scaling group should be added to the EC2 instances that it launches\. The Auto Scaling group applies the tags while the instances are in the `Pending` state\. Note that if you have a lifecycle hook, the tags are available when the instance enters the `Pending:Wait` state\.

Tagging your EC2 instances enables you to see instance cost allocation by tag in your AWS bill\. For more information, see [Using Cost Allocation Tags](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\.


+ [Tag Restrictions](#tag_restrictions)
+ [Tagging Lifecycle](#tag-lifecycle)
+ [Add or Modify Tags for Your Auto Scaling Group](#add-tags)
+ [Delete Tags](#delete-tag)

## Tag Restrictions<a name="tag_restrictions"></a>

The following basic restrictions apply to tags:

+ The maximum number of tags per resource is 50\.

+ The maximum number of tags that you can add or remove using a single call is 25\.

+ The maximum key length is 127 Unicode characters\.

+ The maximum value length is 255 Unicode characters\.

+ Tag keys and values are case sensitive\.

+ Do not use the `aws:` prefix in your tag names or values, because it is reserved for AWS use\. You can't edit or delete tag names or values with this prefix, and they do not count against toward your limit of tags per Auto Scaling group\.

You can add tags to your Auto Scaling group when you create it or when you update it\. You can remove tags from your Auto Scaling group at any time\. For information about assigning tags when you create your Auto Scaling group, see [Step 2: Create an Auto Scaling Group](GettingStartedTutorial.md#gs-create-asg)\.

## Tagging Lifecycle<a name="tag-lifecycle"></a>

If you have opted to propagate tags to your Auto Scaling instances, the tags are managed as follows:

+ When an Auto Scaling group launches instances, it adds the tags to the instances\. In addition, it adds a tag with a key of `aws:autoscaling:groupName` and a value of the name of the Auto Scaling group\.

+ When you attach existing instances, the Auto Scaling group adds the tags to the instances, overwriting any existing tags with the same tag key\. In addition, it adds a tag with a key of `aws:autoscaling:groupName` and a value of the name of the Auto Scaling group\.

+ When you detach an instance from an Auto Scaling group, it removes only the `aws:autoscaling:groupName` tag\.

+ When you scale in manually or the Auto Scaling group automatically scales in, it removes all tags from the instances that are terminating\.

## Add or Modify Tags for Your Auto Scaling Group<a name="add-tags"></a>

When you add a tag to your Auto Scaling group, you can specify whether it should be added to instances launched in the Auto Scaling group\. If you modify a tag, the updated version of the tag is added to instances launched in the Auto Scaling group after the change\. If you create or modify a tag for an Auto Scaling group, these changes are not made to instances that are already running in the Auto Scaling group\.


+ [Add or Modify Tags Using the AWS Management Console](#add-tags-console)
+ [Add or Modify Tags Using the AWS CLI](#add-tags-aws-cli)

### Add or Modify Tags Using the AWS Management Console<a name="add-tags-console"></a>

Use the Amazon EC2 console to add or modify tags\.

**To add or modify tags**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Tags** tab, choose **Add/Edit tags**\. The **Add/Edit Auto Scaling Group Tags** page lists any existing tags for the Auto Scaling group\.

1. To modify existing tags, edit **Key** and **Value**\.

1. To add a new tag, choose **Add tag** and edit **Key** and **Value**\. You can keep **Tag New Instances** selected to add the tag to the instances launched in the Auto Scaling group automatically, and deselect it otherwise\.

1. When you have finished adding tags, choose **Save**\.

### Add or Modify Tags Using the AWS CLI<a name="add-tags-aws-cli"></a>

Use the [create\-or\-update\-tags](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-or-update-tags.html) command to create or modify a tag\. For example, the following command adds a tag with a key of "environment" and a value of "test" that will also be added to instances launched in the Auto Scaling group after this change\. If a tag with this key already exists, the existing tag is replaced\.

```
aws autoscaling create-or-update-tags --tags "ResourceId=my-asg,ResourceType=auto-scaling-group,Key=environment,Value=test,PropagateAtLaunch=true"
```

The following is an example response:

```
OK-Created/Updated tags
```

Use the following [describe\-tags](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-tags.html) command to list the tags for the specified Auto Scaling group\.

```
aws autoscaling describe-tags --filters Name=auto-scaling-group,Values=my-asg
```

The following is an example response:

```
{
    "Tags": [
        {
            "ResourceType": "auto-scaling-group",
            "ResourceId": "my-asg",
            "PropagateAtLaunch": true,
            "Value": "test",
            "Key": "environment"
        }
    ]
}
```

Alternatively, use the following [describe\-auto\-scaling\-groups](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the tag is added to the Auto Scaling group\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
```

The following is an example response:

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupARN": "arn",
            "HealthCheckGracePeriod": 0,
            "SuspendedProcesses": [],
            "DesiredCapacity": 1,
            "Tags": [
                {
                    "ResourceType": "auto-scaling-group",
                    "ResourceId": "my-asg",
                    "PropagateAtLaunch": true,
                    "Value": "test",
                    "Key": "environment"
                }
            ],
            "EnabledMetrics": [],
            "LoadBalancerNames": [],
            "AutoScalingGroupName": "my-asg",
            ...
        }
    ]
}
```

## Delete Tags<a name="delete-tag"></a>

You can delete a tag associated with your Auto Scaling group at any time\.


+ [Delete Tags Using the AWS Management Console](#delete-tag-console)
+ [Delete Tags Using the AWS CLI](#delete-tag-aws-cli)

### Delete Tags Using the AWS Management Console<a name="delete-tag-console"></a>

**To delete a tag using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your Auto Scaling group\.

1. On the **Tags** tab, choose **Add/Edit tags**\. The **Add/Edit Auto Scaling Group Tags** page lists any existing tags for the Auto Scaling group\.

1. Choose the delete icon next to the tag\.

1. Choose **Save**\.

### Delete Tags Using the AWS CLI<a name="delete-tag-aws-cli"></a>

Use the [delete\-tags](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-tags.html) command to delete a tag\. For example, the following command deletes a tag with a key of "environment"\.

```
aws autoscaling delete-tags --tags "ResourceId=my-asg,ResourceType=auto-scaling-group,Key=environment"
```

Notice that you must specify the tag key, but you don't need to specify the value\. If you specify a value and the value is incorrect, the tag is not deleted\.