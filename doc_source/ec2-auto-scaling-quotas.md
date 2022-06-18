# Quotas for Amazon EC2 Auto Scaling<a name="ec2-auto-scaling-quotas"></a>

Your AWS account has default quotas, formerly referred to as limits, for each AWS service\. Unless otherwise noted, each quota is Region\-specific\. You can request increases for some quotas, and other quotas cannot be increased\.

To view the quotas for Amazon EC2 Auto Scaling, open the [Service Quotas console](https://console.aws.amazon.com/servicequotas/home)\. In the navigation pane, choose **AWS services** and select **Amazon EC2 Auto Scaling**\.

To request a quota increase, see [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html) in the *Service Quotas User Guide*\. If the quota is not yet available in Service Quotas, use the [Auto Scaling Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-auto-scaling)\. Quota increases are tied to the Region they were requested for\.

All requests are submitted to AWS Support\. You can track your request case in the AWS Support console\.

**Amazon EC2 Auto Scaling resources**  
Your AWS account has the following quotas related to the number of Auto Scaling groups and launch configurations that you can create\. 


| Resource | Default quota | 
| --- | --- | 
| Auto Scaling groups per region | 500 | 
| Launch configurations per region | 200 | 

**Auto Scaling group configuration**  
Your AWS account has the following quotas related to the configuration of Auto Scaling groups\. They cannot be changed\.


| Resource | Quota | 
| --- | --- | 
| Scaling policies per Auto Scaling group | 50 | 
| Scheduled actions per Auto Scaling group | 125 | 
| Step adjustments per step scaling policy | 20 | 
| Lifecycle hooks per Auto Scaling group | 50 | 
| SNS topics per Auto Scaling group | 10 | 
| Classic Load Balancers per Auto Scaling group | 50 | 
| Target groups per Auto Scaling group | 50 | 

**Auto Scaling group API operations**  
Amazon EC2 Auto Scaling provides API operations to make changes to your Auto Scaling groups in batches\. The following are the API limits on the maximum number of items \(maximum array members\) that are allowed in a single operation\. They cannot be changed\.


| Operation | Maximum array members | 
| --- | --- | 
| [AttachInstances](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachInstances.html) | 20 instance IDs  | 
| [AttachLoadBalancers](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancers.html) | 10 load balancers | 
| [AttachLoadBalancerTargetGroups](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_AttachLoadBalancerTargetGroups.html) | 10 target groups | 
| [BatchDeleteScheduledAction](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_BatchDeleteScheduledAction.html) | 50 scheduled actions | 
| [BatchPutScheduledUpdateGroupAction](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_BatchPutScheduledUpdateGroupAction.html) | 50 scheduled actions | 
| [DetachInstances](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html) | 20 instance IDs | 
| [DetachLoadBalancers](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancers.html) | 10 load balancers | 
| [DetachLoadBalancerTargetGroups](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachLoadBalancerTargetGroups.html) | 10 target groups | 
| [EnterStandby](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_EnterStandby.html) | 20 instance IDs | 
| [ExitStandby](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_ExitStandby.html) | 20 instance IDs | 
| [SetInstanceProtection](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_SetInstanceProtection.html) | 50 instance IDs | 

**Other services**  
Quotas for other services, such as Amazon EC2, can impact your Auto Scaling groups\. For more information, see [Service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/aws-service-information.html) in the *Amazon Web Services General Reference*\.