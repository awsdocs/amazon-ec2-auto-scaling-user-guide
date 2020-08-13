# Troubleshooting Amazon EC2 Auto Scaling: Load balancer issues<a name="ts-as-loadbalancer"></a>

This page provides information about issues caused by the load balancer associated with your Auto Scaling group, potential causes, and the steps you can take to resolve the issues\.

To retrieve an error message, see [Retrieving an error message](CHAP_Troubleshooting.md#RetrievingErrors)\.

When your EC2 instances fail to launch due to issues with the load balancer associated with your Auto Scaling group, you might get one or more of the following error messages\.

**Topics**
+ [Cannot find Load Balancer <your launch environment>\. Validating load balancer configuration failed\.](#ts-as-loadbalancer-1)
+ [There is no ACTIVE Load Balancer named <load balancer name>\. Updating load balancer configuration failed\.](#ts-as-loadbalancer-2)
+ [EC2 instance <instance ID> is not in VPC\. Updating load balancer configuration failed\.](#ts-as-loadbalancer-3)
+ [EC2 instance <instance ID> is in VPC\. Updating load balancer configuration failed\.](#ts-as-loadbalancer-5)
+ [The security token included in the request is invalid\. Validating load balancer configuration failed\.](#ts-as-loadbalancer-4)

## Cannot find Load Balancer <your launch environment>\. Validating load balancer configuration failed\.<a name="ts-as-loadbalancer-1"></a>
+ **Cause 1**: The load balancer has been deleted\.
+ **Solution 1**:

  1. Check to see if your load balancer still exists\. You can use the [https://docs.aws.amazon.com/cli/latest/reference/elb/describe-load-balancers.html](https://docs.aws.amazon.com/cli/latest/reference/elb/describe-load-balancers.html) command\.

  1. If you see your load balancer listed in the response, see **Cause 2**\.

  1. If you do not see your load balancer listed in the response, you can either create a new load balancer and then create a new Auto Scaling group or you can create a new Auto Scaling group without the load balancer\.
+ **Cause 2**: The load balancer name was not specified in the right order when creating the Auto Scaling group\.
+ **Solution 2**: Create a new Auto Scaling group and specify the load balancer name at the end\.

## There is no ACTIVE Load Balancer named <load balancer name>\. Updating load balancer configuration failed\.<a name="ts-as-loadbalancer-2"></a>
+ **Cause**: The specified load balancer might have been deleted\.
+ **Solution**: You can either create a new load balancer and then create a new Auto Scaling group or create a new Auto Scaling group without the load balancer\. 

## EC2 instance <instance ID> is not in VPC\. Updating load balancer configuration failed\.<a name="ts-as-loadbalancer-3"></a>
+ **Cause**: The specified instance does not exist in the VPC\.
+ **Solution**: You can either delete your load balancer associated with the instance or create a new Auto Scaling group\.

## EC2 instance <instance ID> is in VPC\. Updating load balancer configuration failed\.<a name="ts-as-loadbalancer-5"></a>
+ **Cause**: The load balancer is in EC2\-Classic but the Auto Scaling group is in a VPC\.
+ **Solution**: Ensure that the load balancer and the Auto Scaling group are in the same network \(EC2\-Classic or a VPC\)\.

## The security token included in the request is invalid\. Validating load balancer configuration failed\.<a name="ts-as-loadbalancer-4"></a>
+ **Cause**: Your AWS account might have expired\.
+ **Solution**: Check whether your AWS account is valid\. Go to https://aws\.amazon\.com/ and choose **Sign Up Now** to open a new account\.