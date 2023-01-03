# Troubleshoot Amazon EC2 Auto Scaling: Load balancer issues<a name="ts-as-loadbalancer"></a>

This page provides information about issues caused by the load balancer associated with your Auto Scaling group, potential causes, and the steps you can take to resolve the issues\.

To retrieve an error message, see [Retrieve an error message from scaling activities](CHAP_Troubleshooting.md#RetrievingErrors)\.

When your EC2 instances fail to launch due to issues with the load balancer associated with your Auto Scaling group, you might get one or more of the following error messages\.

**Topics**
+ [One or more target groups not found\. Validating load balancer configuration failed\.](#ts-as-loadbalancer-1)
+ [Cannot find Load Balancer <your load balancer>\. Validating load balancer configuration failed\.](#ts-as-loadbalancer-2)
+ [There is no ACTIVE Load Balancer named <load balancer name>\. Updating load balancer configuration failed\.](#ts-as-loadbalancer-3)
+ [EC2 instance <instance ID> is not in VPC\. Updating load balancer configuration failed\.](#ts-as-loadbalancer-4)
+ [EC2 instance <instance ID> is in VPC\. Updating load balancer configuration failed\.](#ts-as-loadbalancer-5)

**Note**  
You can use Reachability Analyzer to troubleshoot connectivity issues by checking whether instances in your Auto Scaling group are reachable through the load balancer\. To learn about the different network misconfiguration issues that are automatically detected by Reachability Analyzer, see [Reachability Analyzer explanation codes](https://docs.aws.amazon.com/vpc/latest/reachability/explanation-codes.html) in the *Reachability Analyzer User Guide*\.

## One or more target groups not found\. Validating load balancer configuration failed\.<a name="ts-as-loadbalancer-1"></a>

**Problem**: When your Auto Scaling group launches instances, Amazon EC2 Auto Scaling tries to validate that the Elastic Load Balancing resources that are associated with the Auto Scaling group exist\. When a target group cannot be found, the scaling activity fails, and you get the `One or more target groups not found. Validating load balancer configuration failed.` error\.

**Cause 1**: A target group attached to your Auto Scaling group has been deleted\.

**Solution 1**: You can either create a new Auto Scaling group without the target group or remove the unused target group from the Auto Scaling group by using the Amazon EC2 Auto Scaling console or the [detach\-load\-balancer\-target\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancer-target-groups.html) command\.

**Cause 2**: The target group exists, but there was an issue trying to specify the target group ARN when creating the Auto Scaling group\. Resources are not created in the right order\.

**Solution 2**: Create a new Auto Scaling group and specify the target group at the end\.

## Cannot find Load Balancer <your load balancer>\. Validating load balancer configuration failed\.<a name="ts-as-loadbalancer-2"></a>

**Problem**: When your Auto Scaling group launches instances, Amazon EC2 Auto Scaling tries to validate that the Elastic Load Balancing resources that are associated with the Auto Scaling group exist\. When a Classic Load Balancer cannot be found, the scaling activity fails, and you get the `Cannot find Load Balancer <your load balancer>. Validating load balancer configuration failed.` error\.

**Cause 1**: The Classic Load Balancer has been deleted\.

**Solution 1**: You can either create a new Auto Scaling group without the load balancer or remove the unused load balancer from the Auto Scaling group by using the Amazon EC2 Auto Scaling console or the [detach\-load\-balancers](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/detach-load-balancers.html) command\.

**Cause 2**: The Classic Load Balancer exists, but there was an issue trying to specify the load balancer name when creating the Auto Scaling group\. Resources are not created in the right order\.

**Solution 2**: Create a new Auto Scaling group and specify the load balancer name at the end\.

## There is no ACTIVE Load Balancer named <load balancer name>\. Updating load balancer configuration failed\.<a name="ts-as-loadbalancer-3"></a>

**Cause**: The specified load balancer might have been deleted\.

**Solution**: You can either create a new load balancer and then create a new Auto Scaling group or create a new Auto Scaling group without the load balancer\. 

## EC2 instance <instance ID> is not in VPC\. Updating load balancer configuration failed\.<a name="ts-as-loadbalancer-4"></a>

**Cause**: The specified instance does not exist in the VPC\.

**Solution**: You can either delete your load balancer associated with the instance or create a new Auto Scaling group\.

## EC2 instance <instance ID> is in VPC\. Updating load balancer configuration failed\.<a name="ts-as-loadbalancer-5"></a>

**Cause**: The load balancer is in EC2\-Classic but the Auto Scaling group is in a VPC\.

**Solution**: Ensure that the load balancer and the Auto Scaling group are in the same network \(EC2\-Classic or a VPC\)\.