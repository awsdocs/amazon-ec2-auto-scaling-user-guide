# Replacing Auto Scaling instances based on an instance refresh<a name="asg-instance-refresh"></a>

You can use an instance refresh to update the instances in your Auto Scaling group instead of manually replacing instances a few at a time\. This can be useful when a configuration change requires you to replace instances, and you have a large number of instances in your Auto Scaling group\. 

An instance refresh can be helpful when you have a new Amazon Machine Image \(AMI\) or a new user data script\. To use an instance refresh, first create a new launch template that specifies the new AMI or user data script\. Then, start an instance refresh to begin updating the instances in the group immediately\. 

## How it works<a name="instance-refresh-how-it-works"></a>

The following example steps show how an instance refresh works: 
+ You create a launch template or a new launch template version with your desired updates\. For more information, see [Creating a launch template for an Auto Scaling group](create-launch-template.md)\.
+ You configure the minimum healthy percentage, instance warmup, and checkpoints, specify your desired configuration that includes your launch template, and start an instance refresh\. The desired configuration can optionally specify whether a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md) is to be applied\.
+ Amazon EC2 Auto Scaling starts performing a rolling replacement of the instances\. It takes a set of instances out of service, terminates them, and launches a set of instances with the new desired configuration\. Then, it waits until the instances pass your health checks and complete warmup before it moves on to replacing other instances\. 
+ After a certain percentage of the group is replaced, a checkpoint is reached\. Whenever there is a checkpoint, Amazon EC2 Auto Scaling temporarily stops replacing instances, sends a notification, and waits for the amount of time you specified before continuing\. After you receive the notification, you can verify that your new instances are working as expected\.
+ After the instance refresh succeeds, the Auto Scaling group settings are automatically updated with the configuration that you specified at the start of the operation\. 

## Core concepts and terms<a name="instance-refresh-core-concepts"></a>

Before you get started, familiarize yourself with the following instance refresh core concepts and terms:

**Minimum healthy percentage**  
As part of starting an instance refresh, you specify the minimum healthy percentage to maintain at all times\. This is the amount of capacity in an Auto Scaling group that must pass your [health checks](healthcheck.md) during an instance refresh so that the operation is allowed to continue\. The value is expressed as a percentage of the desired capacity of the Auto Scaling group \(rounded up to the nearest integer\)\. Setting the minimum healthy percentage to 100 percent limits the rate of replacement to one instance at a time\. In contrast, setting it to 0 percent causes all instances to be replaced at the same time\.

**Instance warmup**  
The *instance warmup* is the time period from when a new instance comes into service to when it can receive traffic\. During an instance refresh, Amazon EC2 Auto Scaling does not immediately move on to the next replacement after determining that a newly launched instance is healthy\. It waits for the warm\-up period that you specified before it moves on to replacing other instances\. This setting can be helpful when you have configuration scripts that take time to run\.

**Desired configuration**  
A *desired configuration* is the new configuration you want Amazon EC2 Auto Scaling to deploy across your Auto Scaling group\. For example, you can specify the launch template and version for your instances\. During an instance refresh, Amazon EC2 Auto Scaling updates the Auto Scaling group to the desired configuration\. If a scale\-out event occurs during an instance refresh, Amazon EC2 Auto Scaling launches new instances with the desired configuration instead of the group's current settings\. After the instance refresh succeeds, Amazon EC2 Auto Scaling updates the settings of the Auto Scaling group to reflect the new desired configuration that you specified as part of the instance refresh\. 

**Skip matching**  
Skip matching means that Amazon EC2 Auto Scaling skips replacing instances that match the desired configuration\. If no desired configuration is specified, then it skips replacing instances that have the same configuration that is already set on the group\. If skip matching is not enabled, any instance in the Auto Scaling group can be replaced with a new instance, regardless of whether there are any updates needed\. 

**Checkpoints**  
A checkpoint is a point in time where the instance refresh pauses for a specified amount of time\. An instance refresh can contain multiple checkpoints\. Amazon EC2 Auto Scaling emits events for each checkpoint, so that you can add an EventBridge rule to send the events to a target such as Amazon SNS to be notified when a checkpoint is reached\. After a checkpoint is reached, you have the opportunity to validate your deployment\. If any problems are identified, you can cancel the instance refresh and then roll it back by initiating another instance refresh\. The ability to deploy updates in phases is a key benefit of checkpoints\. If you don't use checkpoints, rolling replacements are performed continuously\.

## Considerations<a name="instance-refresh-considerations"></a>

The following are things to consider when starting an instance refresh, to help ensure that the group continues to perform as expected\. 
+ While warming up, a newly launched instance is not counted toward the aggregated metrics of the Auto Scaling group\. 
+ If you added scaling policies to the Auto Scaling group, the scaling activities run in parallel\. If you set a long interval for the instance refresh warm\-up period, it takes more time for newly launched instances to be reflected in the metrics\. Therefore, an adequate warm\-up period helps to prevent Amazon EC2 Auto Scaling from scaling on stale metric data\. 
+ If you added a lifecycle hook to the Auto Scaling group, the warm\-up period does not start until the lifecycle hook actions are complete and the instance enters the `InService` state\. 
+ If you enable skip matching but the launch template, the launch template version, and instance types in the mixed instances policy are not changing, the instance refresh will succeed immediately without making any replacements\. If you made any other changes \(for example, changing your Spot allocation strategy\), Amazon EC2 Auto Scaling updates the settings of the Auto Scaling group to reflect the new desired configuration after the instance refresh succeeds\. 

## Start or cancel an instance refresh \(console\)<a name="start-instance-refresh-console"></a>

You can create, view, and cancel your instance refreshes with the Amazon EC2 Auto Scaling console\. If this is your first time starting an instance refresh, using the console will help you understand the features and options available\.

### Starting an instance refresh in the console \(basic procedure\)<a name="starting-an-instance-refresh-in-the-console"></a>

Use the following procedure if you have not previously defined a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md) for your Auto Scaling group\. If you have previously defined a mixed instances policy, see [Starting an instance refresh in the console \(mixed instances group\)](#starting-an-instance-refresh-in-the-console-mig) to start an instance refresh\.

**To start an instance refresh**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. On the **Instance refresh** tab, in **Active instance refresh**, choose **Start instance refresh**\.

1. For **Minimum healthy percentage**, enter the percentage of the Auto Scaling group that must remain healthy during an instance refresh\. The default is 90 percent\. Choosing a lower percentage results in a higher number of instances being terminated and replaced at the same time\. 

1. For **Instance warmup**, enter the number of seconds until a newly launched instance is ready to use\. Set this value greater than or equal to the maximum amount of startup time needed for your application, from when an instance starts to when it can receive traffic\. The default value is the value for the health check grace period that's currently set for the Auto Scaling group\.

1. \(Optional\) For **Checkpoints**, choose **Enable checkpoints** to replace instances using an incremental or phased approach to an instance refresh\. This provides additional time for verification between sets of replacements\. If you choose not to enable checkpoints, the instances are replaced in one nearly continuous operation\.

   If you enable checkpoints, see [Enable checkpoints \(console\)](asg-adding-checkpoints-instance-refresh.md#enable-checkpoints-console) for additional steps\.

1. Enable or turn off **Skip matching**:
   + To skip replacing instances that already match your current launch template, keep the **Enable skip matching** check box selected\.
   + If you turn off skip matching by clearing this check box, all instances can be replaced\.

   When you enable skip matching, instead of using your current launch template, you can set a new launch template or a new version of the current launch template in the **Desired configuration** section of the **Start instance refresh** page\. 
**Note**  
To use the skip matching feature to update an Auto Scaling group that currently uses a launch configuration, you must select a launch template in **Desired configuration**\. Using skip matching with a launch configuration is not supported\.

1. \(Optional\) Expand the **Desired configuration** section to specify any updates you want to make to your Auto Scaling group\.

   From this point forward, you can choose to use JSON or YAML syntax to edit parameter values instead of making selections in the console interface\. To do so, choose **Use code editor** instead of **Use console interface**\. The following procedure explains how to make selections using the console interface\.

   1. For **Update launch template**: 
      + If you *haven't* created a new launch template or a new launch template version for your Auto Scaling group, you can keep this check box unselected\.
      + If you have created a new launch template or a new launch template version, select this check box\. When you select this option, Amazon EC2 Auto Scaling shows you the current launch template and current launch template version, and it lists any other versions that are available\. Choose the launch template and then choose the version\. 

        After you choose a version, you can see the version information\. This is the version of the launch template that will be used when replacing instances as part of an instance refresh\. If the instance refresh succeeds, this version of the launch template will also be used whenever new instances launch, such as when the group scales\.

   1. For **Choose a set of instance types and purchase options to override the instance type in the launch template**:
      + Keep this check box unselected to use the instance type and purchase option you specified in your launch template\.
      + Select this check box if you want to override the instance type in the launch template or take advantage of unused EC2 capacity and save money by running Spot Instances\. When you choose this option, you can either manually add each instance type, or choose a primary instance type and a recommendation option that retrieves any additional matching instance types for you\. If you plan to launch Spot Instances, we recommend adding a few instance types so that Amazon EC2 Auto Scaling can launch another instance type if there is insufficient instance capacity in your chosen Availability Zones\. For further guidance on how to handle these settings, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.
**Warning**  
Spot Instances should not be used with applications that are unable to handle a Spot Instance interruption if the Amazon EC2 Spot service needs to reclaim capacity\.

      If you select this check box, make sure that the launch template doesn't already request Spot Instances\. You cannot use a launch template that requests Spot Instances to create an Auto Scaling group that uses multiple instance types and launches Spot and On\-Demand Instances\.
**Note**  
To configure these options on an Auto Scaling group that currently uses a launch configuration, you must select a launch template in **Update launch template**\. Overriding the instance type in your launch configuration is not supported\.

1. Review all of your selections to confirm that everything is set up correctly\.

   At this point, it's a good idea to verify that the differences between the current and proposed changes won't affect your application in unexpected or unwanted ways\. For example, if you launched your original instances from a paravirtual \(PV\) AMI, but want to change to a current generation instance type that is only supported by a hardware virtual machine \(HVM\) AMI, you need to use an HVM AMI\. For more information, see [Compatibility for changing the instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/resize-limitations.html) in the *Amazon EC2 User Guide for Linux Instances*\.

1. When you are satisfied with your instance refresh selections, choose **Start**\. 

### Starting an instance refresh in the console \(mixed instances group\)<a name="starting-an-instance-refresh-in-the-console-mig"></a>

Use the following procedure if you have created an Auto Scaling group with a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md)\. If you haven't defined a mixed instances policy for your group yet, see [Starting an instance refresh in the console \(basic procedure\)](#starting-an-instance-refresh-in-the-console) to start an instance refresh\.

**To start an instance refresh**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. On the **Instance refresh** tab, in **Active instance refresh**, choose **Start instance refresh**\.

1. For **Minimum healthy percentage**, enter the percentage of the Auto Scaling group that must remain healthy during an instance refresh\. The default value is 90 percent\. Choosing a lower percentage will result in a higher number of instances being terminated and replaced at the same time\. 

1. For **Instance warmup**, enter the number of seconds until a newly launched instance is ready to use\. Set this value greater than or equal to the maximum amount of startup time needed for your application, from when an instance starts to when it can receive traffic\. The default value is the value for the health check grace period that's currently set for the Auto Scaling group\.

1. \(Optional\) For **Checkpoints**, choose **Enable checkpoints** to replace instances using an incremental or phased approach to an instance refresh\. This provides additional time for verification between sets of replacements\. If you choose not to enable checkpoints, the instances are replaced in one nearly continuous operation\.

   If you enable checkpoints, see [Enable checkpoints \(console\)](asg-adding-checkpoints-instance-refresh.md#enable-checkpoints-console) for additional steps\.

1. Enable or turn off **Skip matching**:
   + To skip replacing instances that already match your current launch template and any instance type overrides, keep the **Enable skip matching** check box selected\.
   + If you choose to turn off skip matching by clearing this check box, all instances can be replaced\.

   When you enable skip matching, instead of using your current launch template, you can set a new launch template or a new version of the current launch template in the **Desired configuration** section of the **Start instance refresh** page\. You can also update your instance type overrides in **Desired configuration**\. 

1. In the **Desired configuration** section, do the following\. 

   From this point forward, you can choose to use JSON or YAML syntax to edit parameter values instead of making selections in the console interface\. To do so, choose **Use code editor** instead of **Use console interface**\. The following procedure explains how to make selections using the console interface\.

   1. For **Update launch template**: 
      + If you *haven't* created a new launch template or a new launch template version for your Auto Scaling group, you can keep this check box unselected\.
      + If you have created a new launch template or a new launch template version, select this check box\. When you select this option, Amazon EC2 Auto Scaling shows you the current launch template and current launch template version, and it lists any other versions that are available\. Choose the launch template and then choose the version\. 

        After you choose a version, you can see the version information\. This is the version of the launch template that will be used when replacing instances as part of an instance refresh\. If the instance refresh succeeds, this version of the launch template will also be used whenever new instances launch, such as when the group scales\.

   1. For **Use these settings to override the instance type and purchase option defined in the launch template**: 

      By default, this check box is selected\. Amazon EC2 Auto Scaling populates each parameter with the value that's currently set in the *mixed instances policy* for the Auto Scaling group\. Only update values for the parameters that you want to change\. For guidance on these settings, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.
**Warning**  
We recommend that you do not clear this check box\. Only clear it if you want to stop using a mixed instances policy\. After the instance refresh succeeds, Amazon EC2 Auto Scaling updates your group to match the **Desired configuration**\. If it no longer includes a mixed instances policy, Amazon EC2 Auto Scaling gradually terminates any Spot Instances that are currently running and replaces them with On\-Demand Instances\. Or, if your launch template requests Spot Instances, then Amazon EC2 Auto Scaling gradually terminates any On\-Demand Instances that are currently running and replaces them with Spot Instances\. 

1. Review all of your selections to confirm that everything is set up correctly\.

   At this point, it's a good idea to verify that the differences between the current and proposed changes won't affect your application in unexpected or unwanted ways\. For example, if you launched your original instances from a paravirtual \(PV\) AMI, but want to change to a current generation instance type that is only supported by a hardware virtual machine \(HVM\) AMI, you need to use an HVM AMI\. For more information, see [Compatibility for changing the instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/resize-limitations.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   When you are satisfied with your instance refresh selections, choose **Start**\. 

**To check the status of an instance refresh**

1. On the **Instance refresh** tab, under **Instance refresh history**, you can determine the status of your request by looking at the **Status** column\. The operation goes into `Pending` status while it is initializing\. The status should then quickly change to `InProgress`\. When all instances are updated, the status changes to `Successful`\.

1. On the **Activity** tab, under **Activity history**, when the instance refresh starts, you see entries when instances are terminated and another set of entries when instances are launched\. In the **Description** column, you can find the instance ID\. 

1. On the **Instance management** tab, under **Instances**, you can verify that your instances launched successfully\. Initially, your instances are in the `Pending` state\. After an instance is ready to receive traffic, its state is `InService`\. The **Health status** column shows the result of the health checks on your instances\.

**To cancel an instance refresh**

1. Open the Amazon EC2 Auto Scaling console at [https://console\.aws\.amazon\.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/)\.

1. Select the check box next to the Auto Scaling group\.

1. On the **Instance refresh** tab, in **Active instance refresh**, choose **Cancel instance refresh**\.

1. When prompted for confirmation, choose **Confirm**\.

## Start or cancel an instance refresh \(AWS CLI\)<a name="start-instance-refresh-cli"></a>

**To start an instance refresh**  
Use the following [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command to start an instance refresh from the AWS CLI\. You can specify any preferences that you want to change in a JSON configuration file\. When you reference the configuration file, provide the file's path and name as shown in the following example\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`:

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 400,
      "MinHealthyPercentage": 50
    }
}
```

Alternatively, you can start the instance refresh without the optional preferences by running the following command\. If preferences are not provided, the default values are used for `InstanceWarmup` and `MinHealthyPercentage`\.

```
aws autoscaling start-instance-refresh --auto-scaling-group-name my-asg
```

Example output:

```
{
    "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
}
```

**Note**  
To specify your desired configuration and enable skip matching with the AWS CLI, see the additional start\-instance\-refresh examples in [Instance refresh examples that enable skip matching with the AWS Command Line Interface \(AWS CLI\)](asg-instance-refresh-skip-matching.md)\. 

**To check the status of an instance refresh**  
View the instance refreshes for an Auto Scaling group by using the following [describe\-instance\-refreshes](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-instance-refreshes.html) command\.

```
aws autoscaling describe-instance-refreshes --auto-scaling-group-name my-asg
```

Example output:

```
{
    "InstanceRefreshes": [
        {
            "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b",
            "AutoScalingGroupName": "my-asg",
            "Status": "InProgress",
            "StartTime": "2020-06-02T18:11:27Z",
            "PercentageComplete": 0,
            "InstancesToUpdate": 5
        },
        {
            "InstanceRefreshId": "dd7728d0-5bc4-4575-96a3-1b2c52bf8bb1",
            "AutoScalingGroupName": "my-asg",
            "Status": "Successful",
            "StartTime": "2020-06-02T16:43:19Z",
            "EndTime": "2020-06-02T16:53:37Z",
            "PercentageComplete": 100,
            "InstancesToUpdate": 0
        }
    ]
}
```

**To cancel an instance refresh**  
When you cancel an instance refresh using the [cancel\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/cancel-instance-refresh.html) command from the AWS CLI, specify the name of the Auto Scaling group as shown in the following example\. 

```
aws autoscaling cancel-instance-refresh --auto-scaling-group-name my-asg
```

Example output:

```
{
    "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
}
```

## Limitations<a name="instance-refresh-limitations"></a>
+ **Instances terminated before launch**: When there is only one instance in the Auto Scaling group, starting an instance refresh can result in an outage\. This is because Amazon EC2 Auto Scaling terminates an instance and then launches a new instance\.
+ **Total duration**: The maximum amount of time that an instance refresh can continue to actively replace instances is 14 days\. 
+ **Instances not replaced**: If an instance is on standby or protected from scale in, it cannot be replaced\. If Amazon EC2 Auto Scaling encounters an instance that it cannot replace, it will continue to replace other instances\. 
+ **One\-hour timeout**: When an instance refresh is unable to continue making replacements because your application doesn't pass health checks or there are instances on standby or protected from scale in, it keeps retrying for an hour\. It also provides a status message to help you resolve the issue\. If the problem persists after an hour, the operation fails\. The intention is to give it time to recover if there is a temporary issue\. 
+ **No rollback**: You can cancel an instance refresh at any time, but any instances that have already been replaced are not rolled back to their previous configuration\. If an instance refresh fails, any instances that were already replaced are not rolled back to their previous configuration\. To fix a failed instance refresh, first resolve the underlying issue that caused the update to fail, and then initiate another instance refresh\. 