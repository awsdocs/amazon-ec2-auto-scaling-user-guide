# Logging Amazon EC2 Auto Scaling API Calls By Using AWS CloudTrail<a name="logging-using-cloudtrail"></a>

Amazon EC2 Auto Scaling is integrated with CloudTrail, a service that captures API calls made by or on behalf of Amazon EC2 Auto Scaling in your AWS account and delivers the log files to an Amazon S3 bucket that you specify\. CloudTrail captures API calls from the Amazon EC2 console and from the Amazon EC2 Auto Scaling API\. Using the information collected by CloudTrail, you can determine what request was made to Amazon EC2 Auto Scaling, the source IP address from which the request was made, who made the request, when it was made, and so on\. For more information about CloudTrail, including how to configure and enable it, see the [http://docs.aws.amazon.com/awscloudtrail/latest/userguide/](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Amazon EC2 Auto Scaling Information in CloudTrail<a name="service-name-info-in-cloudtrail"></a>

When CloudTrail logging is enabled in your AWS account, API calls made to Amazon EC2 Auto Scaling actions are tracked in log files\. Amazon EC2 Auto Scaling records are written together with other AWS service records in a log file\. CloudTrail determines when to create and write to a new file based on a time period and file size\.

All of the Amazon EC2 Auto Scaling actions are logged and are documented in the [Amazon EC2 Auto Scaling API Reference](http://docs.aws.amazon.com/autoscaling/ec2/APIReference/)\. For example, calls to the **CreateLaunchConfiguration**, **DescribeAutoScalingGroup**, and **UpdateAutoScalingGroup** actions generate entries in the CloudTrail log files\. 

Every log entry contains information about who generated the request\. The user identity information in the log helps you determine whether the request was made with account or IAM user credentials, with temporary security credentials for a role or federated user, or by another AWS service\. For more information, see **userIdentity** in the [CloudTrail Event Reference](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/event_reference_top_level.html) section in the *AWS CloudTrail User Guide*\.

You can store your log files in your bucket for as long as you want, but you can also define Amazon S3 lifecycle rules to archive or delete log files automatically\. By default, your log files are encrypted by using Amazon S3 server\-side encryption \(SSE\)\.

You can choose to have CloudTrail publish Amazon SNS notifications when new log files are delivered if you want to take quick action upon log file delivery\. For more information, see [Configuring Amazon SNS Notifications](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html) in the *AWS CloudTrail User Guide*\.

You can also aggregate Amazon EC2 Auto Scaling log files from multiple AWS regions and multiple AWS accounts into a single Amazon S3 bucket\. For more information, see [Aggregating CloudTrail Log Files to a Single Amazon S3 Bucket](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/aggregating_logs_top_level.html) in the *AWS CloudTrail User Guide*\.

## Understanding Amazon EC2 Auto Scaling Log File Entries<a name="understanding-service-name-entries"></a>

CloudTrail log files can contain one or more log entries where each entry is made up of multiple JSON\-formatted events\. A log entry represents a single request from any source and includes information about the requested action, any parameters, the date and time of the action, and so on\. The log entries are not guaranteed to be in any particular order\. That is, they are not an ordered stack trace of the public API calls\.

The following example shows a CloudTrail log entry that demonstrates the CreateLaunchConfiguration action\.

```
{
    "Records": [
    {
        "eventVersion": "1.01",
        "userIdentity": {
            "type": "IAMUser",
            "principalId": "EX_PRINCIPAL_ID",
            "arn": "arn:aws:iam::123456789012:user/iamUser1",
            "accountId": "123456789012",
            "accessKeyId": "EXAMPLE_KEY_ID",
            "userName": "iamUser1"
            },
        "eventTime": "2014-06-24T16:53:14Z",
        "eventSource": "autoscaling.amazonaws.com",
        "eventName": "CreateLaunchConfiguration",
        "awsRegion": "us-west-2",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "Amazon CLI/AutoScaling 1.0.61.3 API 2011-01-01",
        "requestParameters": {
            "imageId": "ami-2f726546",
            "instanceType": "m1.small",
            "launchConfigurationName": "launch_configuration_1"
            },
        "responseElements": null,
        "requestID": "07a1becf-fbc0-11e3-bfd8-a5209058e7bb",
        "eventID": "ad30abf7-57db-4a6d-93fa-13deb1fd4cff"
        },
        ...additional entries
    ]
}
```

The following example shows a CloudTrail log entry that demonstrates the DescribeAutoScalingGroups action\.

```
{
    "Records": [
    {
        "eventVersion": "1.01",
        "userIdentity": {
            "type": "IAMUser",
            "principalId": "EX_PRINCIPAL_ID",
            "arn": "arn:aws:iam::123456789012:user/iamUser1",
            "accountId": "123456789012",
            "accessKeyId": "EXAMPLE_KEY_ID",
            "userName": "iamUser1"
            },
        "eventTime": "2014-06-23T23:20:56Z",
        "eventSource": "autoscaling.amazonaws.com",
        "eventName": "DescribeAutoScalingGroups",
        "awsRegion": "us-west-2",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "Amazon CLI/AutoScaling 1.0.61.3 API 2011-01-01",
        "requestParameters": {
            "maxRecords": 20
            },
        "responseElements": null,
        "requestID": "0737e2ea-fb2d-11e3-bfd8-a5209058e7bb",
        "eventID": "0353fb04-281e-47d9-93bb-588bf2256538"
        },
        ...additional entries
    ]
}
```

The following example shows a CloudTrail log entry that demonstrates the UpdateAutoScalingGroups action\.

```
{
    "Records": [
    {
        "eventVersion": "1.01",
        "userIdentity": {
            "type": "IAMUser",
            "principalId": "EX_PRINCIPAL_ID",
            "arn": "arn:aws:iam::123456789012:user/iamUser1",
            "accountId": "123456789012",
            "accessKeyId": "EXAMPLE_KEY_ID",
            "userName": "iamUser1"
            },
        "eventTime": "2014-06-24T16:54:46Z",
        "eventSource": "autoscaling.amazonaws.com",
        "eventName": "UpdateAutoScalingGroup",
        "awsRegion": "us-west-2",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "Amazon CLI/AutoScaling 1.0.61.3 API 2011-01-01",
        "requestParameters": {
            "maxSize": 8,
            "minSize": 1,
            "autoScalingGroupName": "asg1"
            },
        "responseElements": null,
        "requestID": "3ed07c03-fbc0-11e3-bfd8-a5209058e7bb",
        "eventID": "b52ca0aa-5199-4873-a546-55f7c896a4ce"
        },
        ...additional entries
    ]

}
```