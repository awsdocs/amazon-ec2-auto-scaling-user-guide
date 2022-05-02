# Tag Auto Scaling groups and instances<a name="ec2-auto-scaling-tagging"></a>

A *tag* is a custom attribute label that you assign or that AWS assigns to an AWS resource\. Each tag has two parts: 
+ A tag key \(for example, `costcenter`, `environment`, or `project`\)
+ An optional field known as a tag value \(for example, `111122223333` or `production`\)

Tags help you do the following:
+ Track your AWS costs\. You activate these tags on the AWS Billing and Cost Management dashboard\. AWS uses the tags to categorize your costs and deliver a monthly cost allocation report to you\. For more information, see [Using cost allocation tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing User Guide*\.
+ Filter and search for Auto Scaling groups based on the tags you add\. For more information, see [Use tags to filter Auto Scaling groups](#use-tag-filters-aws-cli)\.
+ Control access to Auto Scaling groups based on tags\. You can use conditions in your IAM policies to control access to Auto Scaling groups based on the tags on that group\. For more information, see [Tags for security](#tag-security)\.
+ Identify and organize your AWS resources\. Many AWS services support tagging, so you can assign the same tag to resources from different services to indicate that the resources are related\.

You can tag new or existing Auto Scaling groups\. You can also propagate tags from an Auto Scaling group to the Amazon EC2 instances it launches\. 

Tags are not propagated to Amazon EBS volumes\. To add tags to Amazon EBS volumes, specify the tags in a launch template\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\.

You can create and manage tags through the AWS Management Console, AWS CLI, or SDKs\. 

**Topics**
+ [Tag naming and usage restrictions](#tag_restrictions)
+ [EC2 instance tagging lifecycle](#tag-lifecycle)
+ [Tag your Auto Scaling groups](#add-tags)
+ [Delete tags](#delete-tag)
+ [Tags for security](#tag-security)
+ [Control access to tags](#tag-permissions)
+ [Use tags to filter Auto Scaling groups](#use-tag-filters-aws-cli)

## Tag naming and usage restrictions<a name="tag_restrictions"></a>

The following basic restrictions apply to tags:
+ The maximum number of tags per resource is 50\.
+ The maximum number of tags that you can add or remove using a single call is 25\.
+ The maximum key length is 128 Unicode characters\.
+ The maximum value length is 256 Unicode characters\.
+ Tag keys and values are case\-sensitive\. As a best practice, decide on a strategy for capitalizing tags, and consistently implement that strategy across all resource types\. 
+ Do not use the `aws:` prefix in your tag names or values, because it is reserved for AWS use\. You can't edit or delete tag names or values with this prefix, and they do not count toward your tags per resource quota\.

## EC2 instance tagging lifecycle<a name="tag-lifecycle"></a>

If you have opted to propagate tags to your Amazon EC2 instances, the tags are managed as follows:
+ When an Auto Scaling group launches instances, it adds tags to the instances during resource creation rather than after the resource is created\. 
+ The Auto Scaling group automatically adds a tag to instances with a key of `aws:autoscaling:groupName` and a value of the Auto Scaling group name\. 
+ If you specify instance tags in your launch template and you opted to propagate your group's tags to its instances, all the tags are merged\. If the same tag key is specified for a tag in your launch template and a tag in your Auto Scaling group, then the tag value from the group takes precedence\.
+ When you attach existing instances, the Auto Scaling group adds the tags to the instances, overwriting any existing tags with the same tag key\. It also adds a tag with a key of `aws:autoscaling:groupName` and a value of the Auto Scaling group name\.
+ When you detach an instance from an Auto Scaling group, it removes only the `aws:autoscaling:groupName` tag\.

## Tag your Auto Scaling groups<a name="add-tags"></a>

When you add a tag to your Auto Scaling group, you can specify whether it should be added to instances launched in the Auto Scaling group\. If you modify a tag, the updated version of the tag is added to instances launched in the Auto Scaling group after the change\. If you create or modify a tag for an Auto Scaling group, these changes are not made to instances that are already running in the Auto Scaling group\.

**Topics**
+ [Add or modify tags \(console\)](#add-tags-console)
+ [Add or modify tags \(AWS CLI\)](#add-tags-aws-cli)

### Add or modify tags \(console\)<a name="add-tags-console"></a>

**To tag an Auto Scaling group on creation**  
When you use the Amazon EC2 console to create an Auto Scaling group, you can specify tag keys and values on the **Add tags** page of the Create Auto Scaling group wizard\. To propagate a tag to the instances launched in the Auto Scaling group, make sure that you keep the **Tag new instances** option for that tag selected\. Otherwise, you can deselect it\. 

**To add or modify tags for an existing Auto Scaling group**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\. 

1. On the **Details** tab, choose **Tags**, **Edit**\.

1. To modify existing tags, edit **Key** and **Value**\.

1. To add a new tag, choose **Add tag** and edit **Key** and **Value**\. You can keep **Tag new instances** selected to add the tag to the instances launched in the Auto Scaling group automatically, and deselect it otherwise\.

1. When you have finished adding tags, choose **Update**\.

### Add or modify tags \(AWS CLI\)<a name="add-tags-aws-cli"></a>

The following examples show how to use the AWS CLI to add tags when you create Auto Scaling groups, and to add or modify tags for existing Auto Scaling groups\. 

**To tag an Auto Scaling group on creation**  
Use the [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create a new Auto Scaling group and add a tag, for example, **environment=production**, to the Auto Scaling group\. The tag is also added to any instances launched in the Auto Scaling group\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-configuration-name my-launch-config --min-size 1 --max-size 3 \
  --vpc-zone-identifier "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782" \
  --tags Key=environment,Value=production,PropagateAtLaunch=true
```

**To create or modify tags for an existing Auto Scaling group**  
Use the [create\-or\-update\-tags](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-or-update-tags.html) command to create or modify a tag\. For example, the following command adds the `Name=my-asg` and `costcenter=cc123` tags\. The tags are also added to any instances launched in the Auto Scaling group after this change\. If a tag with either key already exists, the existing tag is replaced\. The Amazon EC2 console associates the display name for each instance with the name that is specified for the `Name` key \(case\-sensitive\)\.

```
aws autoscaling create-or-update-tags \
  --tags ResourceId=my-asg,ResourceType=auto-scaling-group,Key=Name,Value=my-asg,PropagateAtLaunch=true \
  ResourceId=my-asg,ResourceType=auto-scaling-group,Key=costcenter,Value=cc123,PropagateAtLaunch=true
```

#### Describe the tags for an Auto Scaling group \(AWS CLI\)<a name="describe-tags-aws-cli"></a>

If you want to view the tags that are applied to a specific Auto Scaling group, you can use either of the following commands: 
+ [describe\-tags](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-tags.html) — You supply your Auto Scaling group name to view a list of the tags for the specified group\.

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
              "Value": "production",
              "Key": "environment"
          }
      ]
  }
  ```
+ [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) — You supply your Auto Scaling group name to view the attributes of the specified group, including any tags\.

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
                      "Value": "production",
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

## Delete tags<a name="delete-tag"></a>

You can delete a tag associated with your Auto Scaling group at any time\.

**Topics**
+ [Delete tags \(console\)](#delete-tag-console)
+ [Delete tags \(AWS CLI\)](#delete-tag-aws-cli)

### Delete tags \(console\)<a name="delete-tag-console"></a>

**To delete a tag**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to an existing group\.

   A split pane opens up in the bottom of the **Auto Scaling groups** page\.

1. On the **Details** tab, choose **Tags**, **Edit**\.

1. Choose **Remove** next to the tag\.

1. Choose **Update**\.

### Delete tags \(AWS CLI\)<a name="delete-tag-aws-cli"></a>

Use the [delete\-tags](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/delete-tags.html) command to delete a tag\. For example, the following command deletes a tag with a key of `environment`\.

```
aws autoscaling delete-tags --tags "ResourceId=my-asg,ResourceType=auto-scaling-group,Key=environment"
```

You must specify the tag key, but you don't have to specify the value\. If you specify a value and the value is incorrect, the tag is not deleted\.

## Tags for security<a name="tag-security"></a>

IAM supports controlling access to Auto Scaling groups based on tags\. To control access based on tags, provide tag information in the condition element of an IAM policy\. 

For example, you could deny access to all Auto Scaling groups that include a tag with the key `environment` and the value `production`, as shown in the following example\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [        
                "autoscaling:CreateAutoScalingGroup",
                "autoscaling:UpdateAutoScalingGroup",
                "autoscaling:DeleteAutoScalingGroup"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {"autoscaling:ResourceTag/environment": "production"}
            }
        }
    ]
}
```

For details, see [Authorization based on Amazon EC2 Auto Scaling tags](control-access-using-iam.md#security_iam_service-with-iam-tags)\.

For more examples of IAM policies based on tags, see [Amazon EC2 Auto Scaling identity\-based policy examples](security_iam_id-based-policy-examples.md)\.

## Control access to tags<a name="tag-permissions"></a>

IAM also supports controlling which IAM users and groups in your account have permissions to add, modify, or delete tags for Auto Scaling groups\. To control access to tags, provide tag information in the condition element of an IAM policy\. 

For example, you could create an IAM policy that allows removing only the tag with the `temporary` key from Auto Scaling groups\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [        
                "autoscaling:DeleteTags"
            ],
            "Resource": "*",
            "Condition": {
                "ForAllValues:StringEquals": { "aws:TagKeys": ["temporary"] }
            }
        }
    ]
}
```

For more examples of IAM policies based on tags, see [Amazon EC2 Auto Scaling identity\-based policy examples](security_iam_id-based-policy-examples.md)\.

**Note**  
Even if you have a policy that restricts your users from performing a tagging \(or untagging\) operation on an Auto Scaling group, this does not prevent them from manually changing the tags on the instances after they have launched\. For examples that control access to tags on EC2 instances, see [Example: Tagging resources](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html#iam-example-taggingresources) in the *Amazon EC2 User Guide for Linux Instances*\.

## Use tags to filter Auto Scaling groups<a name="use-tag-filters-aws-cli"></a>

The following examples show you how to use filters with the [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to describe Auto Scaling groups with specific tags\. Filtering by tags is limited to the AWS CLI or an SDK, and is not available from the console\.

**Filtering considerations**
+ You can specify multiple filters and multiple filter values in a single request\.
+ You cannot use wildcards with the filter values\. 
+ Filter values are case\-sensitive\.

**Example: Describe Auto Scaling groups with a specific tag key and value pair**  
The following command shows how to filter results to show only Auto Scaling groups with the tag key and value pair of **`environment=production`**\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag-key,Values=environment Name=tag-value,Values=production
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
                    "Value": "production",
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

Alternatively, you can specify tags using a `tag:<key>` filter\. For example, the following command shows how to filter results to show only Auto Scaling groups with a tag key and value pair of **`environment=production`**\. This filter is formatted as follows: `Name=tag:<key>,Values=<value>`, with **<key>** and **<value>** representing a tag key and value pair\. 

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag:environment,Values=production
```

You can also filter AWS CLI output by using the `--query` option\. The following example shows how to limit AWS CLI output for the previous command to the group name, minimum size, maximum size, and desired capacity attributes only\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag:environment,Values=production \
  --query "AutoScalingGroups[].{AutoScalingGroupName: AutoScalingGroupName, MinSize: MinSize, MaxSize: MaxSize, DesiredCapacity: DesiredCapacity}"
```

The following is an example response\.

```
[
    {
        "AutoScalingGroupName": "my-asg",
        "MinSize": 0,
        "MaxSize": 10,
        "DesiredCapacity": 1
    }
  ...
]
```

For more information about filtering, see [Filtering AWS CLI output](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html) in the *AWS Command Line Interface User Guide*\.

**Example: Describe Auto Scaling groups with tags that match the tag key specified**  
The following command shows how to filter results to show only Auto Scaling groups with the `environment` tag, regardless of the tag value\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag-key,Values=environment
```

**Example: Describe Auto Scaling groups with tags that match the set of tag keys specified**  
The following command shows how to filter results to show only Auto Scaling groups with tags for `environment` and `project`, regardless of the tag values\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag-key,Values=environment Name=tag-key,Values=project
```

**Example: Describe Auto Scaling groups with tags that match at least one of the tag keys specified**  
The following command shows how to filter results to show only Auto Scaling groups with tags for `environment` or `project`, regardless of the tag values\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag-key,Values=environment,project
```

**Example: Describe Auto Scaling groups with the specified tag value**  
The following command shows how to filter results to show only Auto Scaling groups with a tag value of `production`, regardless of the tag key\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag-value,Values=production
```

**Example: Describe Auto Scaling groups with the set of tag values specified**  
The following command shows how to filter results to show only Auto Scaling groups with the tag values `production` and `development`, regardless of the tag key\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag-value,Values=production Name=tag-value,Values=development
```

**Example: Describe Auto Scaling groups with tags that match at least one of the tag values specified**  
The following command shows how to filter results to show only Auto Scaling groups with a tag value of `production` or `development`, regardless of the tag key\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag-value,Values=production,development
```

**Example: Describe Auto Scaling groups with tags that match multiple tag keys and values**  
You can also combine filters to create custom AND and OR logic to do more complex filtering\.

The following command shows how to filter results to show only Auto Scaling groups with a specific set of tags\. One tag key is `environment` AND the tag value is \(`production` OR `development`\) AND the other tag key is `costcenter` AND the tag value is `cc123`\.

```
aws autoscaling describe-auto-scaling-groups \
  --filters Name=tag:environment,Values=production,development Name=tag:costcenter,Values=cc123
```