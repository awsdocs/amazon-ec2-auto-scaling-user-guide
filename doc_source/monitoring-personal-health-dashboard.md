# Personal Health Dashboard notifications for Amazon EC2 Auto Scaling<a name="monitoring-personal-health-dashboard"></a>

Your Personal Health Dashboard \(PHD\) provides support for notifications that come from Amazon EC2 Auto Scaling\. These notifications provide awareness and remediation guidance for resource performance or availability issues that may affect your applications\. Only events that are specific to missing security groups and launch templates are currently available\. 

The Personal Health Dashboard is part of the AWS Health service\. It requires no set up and can be viewed by any user that is authenticated in your account\. For more information, see [Getting started with the AWS Personal Health Dashboard](https://docs.aws.amazon.com/health/latest/ug/getting-started-phd.html)\. 

If you receive a message similar to the following messages, it should be treated as an alarm to take action\.

**Example: Auto Scaling group is not scaling out due to a missing security group**

```
  Hello,

  At 2020-01-11 04:00 UTC, we detected an issue with your Auto Scaling group [ARN] in
  AWS account 123456789012.

  A security group associated with this Auto Scaling group cannot be found. Each time a 
  scale out operation is performed, it will be prevented until you make a change that 
  fixes the issue.

  We recommend that you review and update your Auto Scaling group configuration to change 
  the launch template or launch configuration that depends on the unavailable security 
  group.
        
  Sincerely, 
  Amazon Web Services
```

**Example: Auto Scaling group is not scaling out due to a missing launch template**

```
  Hello,  

  At 2020-01-11 04:00 UTC, we detected an issue with your Auto Scaling group [ARN] in 
  AWS account 123456789012.

  The launch template associated with this Auto Scaling group cannot be found. Each time 
  a scale out operation is performed, it will be prevented until you make a change that 
  fixes the issue.

  We recommend that you review and update your Auto Scaling group configuration and 
  specify an existing launch template to use.
        
  Sincerely, 
  Amazon Web Services
```