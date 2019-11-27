# Tagging Auto Scaling Groups and Instances<a name="autoscaling-tagging"></a>

You can organize and manage your Auto Scaling groups by assigning your own metadata to each group as *tags*\. You specify a *key* and a *value* for each tag\. A key can be a general category, such as "project," "owner," or "environment," with specific associated values\. For example, to differentiate between your testing and production environments, you could assign each Auto Scaling group a tag with a key of "environment\." Use a value of "test" to indicate your test environment or "production" to indicate your production environment\. We recommend that you use a consistent set of tags to assist you in tracking your Auto Scaling groups\. 

Additionally, you can propagate the tags from the Auto Scaling group to the Amazon EC2 instances it launches\. Tagging your instances enables you to see instance cost allocation by tag in your AWS bill\. For more information, see [Using Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\.

You can add one or more tags to an Auto Scaling group when you create it\. You can also add, list, edit, or delete tags for existing Auto Scaling groups\. 

You can also control which IAM users and groups in your account have permission to create, edit, or delete tags\. For more information, see [Controlling Access to Your Amazon EC2 Auto Scaling Resources](control-access-using-iam.md)\. Keep in mind, however, that a policy that restricts your users from performing a tagging operation on an Auto Scaling group does not prevent them from manually changing the tags on the instances after they have launched\. For information about IAM policies for Amazon EC2, see [Controlling Access to Amazon EC2 Resources](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/UsingIAM.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**Important**  
You can also add tags to instances by specifying the tags in your launch template\. However, use caution and ensure that you do not use duplicate keys for instance tags\. If you do so, Amazon EC2 Auto Scaling overrides the tag value from your launch template with the value for the same key specified by the Auto Scaling group\. For information about specifying tags in a launch template, see [Creating a Launch Template for an Auto Scaling Group](create-launch-template.md)\. By specifying the tags in a launch template, you can also add tags to Amazon EBS volumes on creation\. 

**Topics**
+ [Tag Restrictions](#tag_restrictions)
+ [Tagging Lifecycle](#tag-lifecycle)
+ [Add or Modify Tags for Your Auto Scaling Group](#add-tags)
+ [Delete Tags](#delete-tag)

## Tag Restrictions<a name="tag_restrictions"></a>

The following basic restrictions apply to tags:
+ The maximum number of tags per resource is 50\.
+ The maximum number of tags that you can add or remove using a single call is 25\.
+ The maximum key length is 128 Unicode characters\.
+ The maximum value length is 256 Unicode characters\.
+ Tag keys and values are case\-sensitive\.
+ Do not use the `aws:` prefix in your tag names or values, because it is reserved for AWS use\. You can't edit or delete tag names or values with this prefix, and they do not count toward your limit of tags per Auto Scaling group\.

## Tagging Lifecycle<a name="tag-lifecycle"></a>

If you have opted to propagate tags to your Amazon EC2 instances, the tags are managed as follows:
+ In most cases, when an Auto Scaling group launches instances, it adds tags to the instances during resource creation rather than after the resource is created\. 
  + The exception is when you use a launch configuration to launch Spot Instances\. For this scenario, your Auto Scaling group adds tags while the instances are in the `Pending` lifecycle state\. If you have a lifecycle hook, the tags are available when the instance enters the `Pending:Wait` lifecycle state\. For more information, see [Auto Scaling Lifecycle](AutoScalingGroupLifecycle.md)\. If you need the Auto Scaling group to add tags to instances as part of the same API call that launches the Spot Instances, consider migrating to launch templates\. For more information, see [Launch Templates](LaunchTemplates.md)\. 
+ The Auto Scaling group automatically adds a tag to the instances with a key of `aws:autoscaling:groupName` and a value of the name of the Auto Scaling group\. 
+ When you attach existing instances, the Auto Scaling group adds the tags to the instances, overwriting any existing tags with the same tag key\. In addition, it adds a tag with a key of `aws:autoscaling:groupName` and a value of the name of the Auto Scaling group\.
+ When you detach an instance from an Auto Scaling group, it removes only the `aws:autoscaling:groupName` tag\.
+ When you scale in manually or the Auto Scaling group automatically scales in, it removes all tags from the instances that are terminating\.

## Add or Modify Tags for Your Auto Scaling Group<a name="add-tags"></a>

When you add a tag to your Auto Scaling group, you can specify whether it should be added to instances launched in the Auto Scaling group\. If you modify a tag, the updated version of the tag is added to instances launched in the Auto Scaling group after the change\. If you create or modify a tag for an Auto Scaling group, these changes are not made to instances that are already running in the Auto Scaling group\.

**Topics**
+ [Add or Modify Tags \(Console\)](#add-tags-console)
+ [Add or Modify Tags \(AWS CLI\)](#add-tags-aws-cli)

### Add or Modify Tags \(Console\)<a name="add-tags-console"></a>

Use the Amazon EC2 console to:
+ Add tags to new Auto Scaling groups when you create them
+ Add, modify, or delete tags for existing Auto Scaling groups

**To tag an Auto Scaling group on creation**  
When you use the Amazon EC2 console to create an Auto Scaling group, you can specify tag keys and values on the **Configure Tags** page of the Create Auto Scaling Group wizard\. To propagate a tag to the instances launched in the Auto Scaling group, make sure that you keep the **Tag New Instances** option for that tag selected\. Otherwise, you can deselect it\. 

**To add or modify tags for an existing Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Choose an existing group from the list\.

1. On the **Tags** tab, choose **Add/Edit tags**\. The **Add/Edit Auto Scaling Group Tags** page lists any existing tags for the Auto Scaling group\.

1. To modify existing tags, edit **Key** and **Value**\.

1. To add a new tag, choose **Add tag** and edit **Key** and **Value**\. You can keep **Tag New Instances** selected to add the tag to the instances launched in the Auto Scaling group automatically, and deselect it otherwise\.

1. When you have finished adding tags, choose **Save**\.

### Add or Modify Tags \(AWS CLI\)<a name="add-tags-aws-cli"></a>

The following examples show how to use the AWS CLI to add tags when you create Auto Scaling groups, and to add or modify tags for existing Auto Scaling groups\. 

**To tag an Auto Scaling group on creation**
+ Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create a new Auto Scaling group and add a tag, for example, `env=prod`, to the Auto Scaling group\. The tag is also added to any instances launched in the Auto Scaling group\.

  ```
  aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
    --launch-configuration-name my-launch-config --min-size 1 --max-size 3 \
    --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
    --tags Key=env,Value=prod,PropagateAtLaunch=true
  ```

**To create or modify tags for an existing Auto Scaling group**
+ Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-or-update-tags.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-or-update-tags.html) command to create or modify a tag\. For example, the following command adds the `Name=my-asg` and `cost-center=cc123` tags\. The tags are also added to any instances launched in the Auto Scaling group after this change\. If a tag with either key already exists, the existing tag is replaced\. The Amazon EC2 console associates the display name for each instance with the name that is specified for the `Name` key \(case\-sensitive\)\.

  ```
  aws autoscaling create-or-update-tags \
    --tags ResourceId=my-asg,ResourceType=auto-scaling-group,Key=Name,Value=my-asg,PropagateAtLaunch=true \
    ResourceId=my-asg,ResourceType=auto-scaling-group,Key=cost-center,Value=cc123,PropagateAtLaunch=true
  ```

**To list all tags for an Auto Scaling group**
+ Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-tags.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-tags.html) command to list the tags for the specified Auto Scaling group\.

  ```
  aws autoscaling describe-tags --filters Name=auto-scaling-group,Values=my-asg
  ```

  The following is an example response\.

  ```
  {
      "Tags": [
          {
              "ResourceType": "auto-scaling-group",
              "ResourceId": "my-asg",
              "PropagateAtLaunch": true,
              "Value": "prod",
              "Key": "env"
          }
      ]
  }
  ```
+ Alternatively, use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the tag is added to the Auto Scaling group\.

  ```
  aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
  ```

  The following is an example response\.

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
                      "Value": "prod",
                      "Key": "env"
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

**Topics**
+ [Delete Tags \(Console\)](#delete-tag-console)
+ [Delete Tags \(AWS CLI\)](#delete-tag-aws-cli)

### Delete Tags \(Console\)<a name="delete-tag-console"></a>

**To delete a tag**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Choose an existing group from the list\.

1. On the **Tags** tab, choose **Add/Edit tags**\. The **Add/Edit Auto Scaling Group Tags** page lists any existing tags for the Auto Scaling group\.

1. Choose the delete icon next to the tag\.

1. Choose **Save**\.

### Delete Tags \(AWS CLI\)<a name="delete-tag-aws-cli"></a>

Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-tags.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-tags.html) command to delete a tag\. For example, the following command deletes a tag with a key of `env`\.

```
aws autoscaling delete-tags --tags "ResourceId=my-asg,ResourceType=auto-scaling-group,Key=env"
```

You must specify the tag key, but you don't have to specify the value\. If you specify a value and the value is incorrect, the tag is not deleted\.