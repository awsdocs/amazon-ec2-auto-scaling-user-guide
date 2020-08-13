# Getting recommendations for an instance type<a name="asg-getting-recommendations"></a>

AWS provides Amazon EC2 instance recommendations to help you improve performance, save money, or both, by using features powered by AWS Compute Optimizer\. You can use these recommendations to decide whether to move to a new instance type\. 

To make recommendations, Compute Optimizer analyzes your existing instance specifications and recent metric history\. The compiled data is then used to recommend which Amazon EC2 instance types are best optimized to handle the existing performance workload\. Recommendations are returned along with per\-hour instance pricing\. 

**Note**  
To get recommendations from Compute Optimizer, you must first opt in to Compute Optimizer\. For more information, see [Getting started with AWS AWS Compute Optimizer](https://docs.aws.amazon.com/compute-optimizer/latest/ug/getting-started.html) in the *AWS Compute Optimizer User Guide*\. 

**Topics**
+ [Limitations](#compute-optimizer-limitations)
+ [Findings](#findings-classifications)
+ [Viewing recommendations](#viewing-recommendations)
+ [Considerations for evaluating the recommendations](#considerations)

## Limitations<a name="compute-optimizer-limitations"></a>

Compute Optimizer currently generates recommendations for M, C, R, T, and X instance types\. Other instance types are not considered by Compute Optimizer\. When you use other instance types, they are excluded from the recommendations\. 

Compute Optimizer currently generates recommendations for Auto Scaling groups that have the same values for desired, minimum, and maximum capacity, and that are configured to launch a single instance type\. 

## Findings<a name="findings-classifications"></a>

Compute Optimizer classifies its findings for Auto Scaling groups as follows:
+ **Not optimized** – An Auto Scaling group is considered not optimized when Compute Optimizer has identified a recommendation that can provide better performance for your workload\. 
+ **Optimized** – An Auto Scaling group is considered optimized when Compute Optimizer determines that the group is correctly provisioned to run your workload, based on the chosen instance type\. For optimized resources, Compute Optimizer might sometimes recommend a new generation instance type\. 
+ **None** – There are no recommendations for this Auto Scaling group\. This might occur if you've been opted in to Compute Optimizer for less than 12 hours, or when the Auto Scaling group has been running for less than 30 hours, or when the Auto Scaling group or instance type is not supported by Compute Optimizer\. For more information, see [Limitations](#compute-optimizer-limitations) in the previous section\.

## Viewing recommendations<a name="viewing-recommendations"></a>

After you opt in to Compute Optimizer, you can view the findings and recommendations that it generates for your Auto Scaling groups\. If you recently opted in, recommendations might not be available for up to 12 hours\.

**To view recommendations generated for an Auto Scaling group**

1. Open the Compute Optimizer console at [https://console\.aws\.amazon\.com/compute\-optimizer/](https://console.aws.amazon.com/compute-optimizer/)\. 

   The Dashboard page opens\.

1. Choose **View recommendations for all Auto Scaling groups**\.

1. Select your Auto Scaling group\.

1. Choose **View detail**\.

   The view changes to display up to three different instance recommendations in a preconfigured view, based on default table settings\. It also provides recent CloudWatch metric data \(average CPU utilization, average network in, and average network out\) for the Auto Scaling group\.

Determine whether you want to use one of the recommendations\. Decide whether to optimize for performance improvement, for cost reduction, or for a combination of these two\. 

To change the instance type in your Auto Scaling group, update the launch template or update the Auto Scaling group to use a new launch configuration\. Existing instances continue to use the previous configuration\. To update the existing instances, terminate them so that they are replaced by your Auto Scaling group, or allow automatic scaling to gradually replace older instances with newer instances based on your [termination policies](as-instance-termination.md)\. 

**Note**  
With the maximum instance lifetime and instance refresh features, you can also replace existing instances in your Auto Scaling group to launch new instances that use the new launch template or launch configuration\. For more information, see [Replacing Auto Scaling instances based on maximum instance lifetime](asg-max-instance-lifetime.md) and [Replacing Auto Scaling instances based on an instance refresh](asg-instance-refresh.md)\.

## Considerations for evaluating the recommendations<a name="considerations"></a>

Before moving to a new instance type, consider the following:
+ The recommendations don't forecast your usage\. Recommendations are based on your historical usage over the most recent 14\-day time period\. Be sure to choose an instance type that is expected to meet your future usage needs\. 
+ Focus on the graphed metrics to determine whether actual usage is lower than instance capacity\. You can also view metric data \(average, peak, percentile\) in CloudWatch to further evaluate your EC2 instance recommendations\. For example, notice how CPU percentage metrics change during the day and whether there are peaks that need to be accommodated\. For more information, see [Viewing available metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/viewing_metrics_with_cloudwatch.html) in the *Amazon CloudWatch User Guide*\.
+ Compute Optimizer might supply recommendations for burstable performance instances, which are T3, T3a, and T2 instances\. If you periodically burst above your baseline, make sure that you can continue to do so based on the vCPUs of the new instance type\. For more information, see [CPU credits and baseline performance for burstable performance instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-credits-baseline-concepts.html) in the *Amazon EC2 User Guide for Linux Instances*\.
+ If you've purchased a Reserved Instance, your On\-Demand Instance might be billed as a Reserved Instance\. Before you change your current instance type, first evaluate the impact on Reserved Instance utilization and coverage\. 
+ Consider conversions to newer generation instances, where possible\.
+ When migrating to a different instance family, make sure the current instance type and the new instance type are compatible, for example, in terms of virtualization, architecture, or network type\. For more information, see [Compatibility for resizing instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html#resize-limitations) in the *Amazon EC2 User Guide for Linux Instances*\.
+ Finally, consider the performance risk rating that's provided for each recommendation\. Performance risk indicates the amount of effort you might need to spend in order to validate whether the recommended instance type meets the performance requirements of your workload\. We also recommend rigorous load and performance testing before and after making any changes\.

**Additional resources**  
In addition to the topics on this page, see the following resources: 
+ [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
+ [AWS Compute Optimizer User Guide](https://docs.aws.amazon.com/compute-optimizer/latest/ug)