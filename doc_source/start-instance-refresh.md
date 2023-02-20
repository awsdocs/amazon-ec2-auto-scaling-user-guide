# Start an instance refresh<a name="start-instance-refresh"></a>

**Important**  
You can roll back an instance refresh that is in progress to undo any changes\. For this to work, the Auto Scaling group must meet the prerequisites for using rollbacks before starting the instance refresh\. For more information, see [Undo changes with a rollback](instance-refresh-rollback.md)\.

The following procedures help you start an instance refresh using the AWS Management Console or AWS CLI\.

## Start an instance refresh \(console\)<a name="start-instance-refresh-console"></a>

If this is your first time starting an instance refresh, using the console will help you understand the features and options available\.

### Start an instance refresh in the console \(basic procedure\)<a name="starting-an-instance-refresh-in-the-console"></a>

Use the following procedure if you have not previously defined a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md) for your Auto Scaling group\. If you have previously defined a mixed instances policy, see [Start an instance refresh in the console \(mixed instances group\)](#starting-an-instance-refresh-in-the-console-mig) to start an instance refresh\.

**To start an instance refresh**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. On the **Instance refresh** tab, in **Active instance refresh**, choose **Start instance refresh**\.

1. For **Minimum healthy percentage**, enter the percentage of the Auto Scaling group that must remain healthy during an instance refresh\. Setting the minimum healthy percentage to 100 percent limits the rate of replacement to one instance at a time\. In contrast, setting it to 0 percent makes it replace all the instances at the same time\. 

1. For **Instance warmup**, enter the number of seconds from when a new instance's state changes to `InService` to when it can receive traffic\. Amazon EC2 Auto Scaling waits this amount of time before moving on to replace the next instance\. 

   While warming up, a newly launched instance is also not counted toward the aggregated instance metrics of the Auto Scaling group \(such as `CPUUtilization`, `NetworkIn`, and `NetworkOut`\)\. If you added scaling policies to the Auto Scaling group, the scaling activities run in parallel\. If you set a long interval for the instance refresh warm\-up period, it takes more time for newly launched instances to show up in the metrics\. Therefore, an adequate warm\-up period keeps Amazon EC2 Auto Scaling from scaling on stale metric data\.

   If you have already correctly defined a default instance warmup for the Auto Scaling group, then you don't need to change the instance warmup\. However, if you want to override the default, you can\. For more information about setting the default instance warmup, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.

1. \(Optional\) For **Checkpoints**, choose **Enable checkpoints** to replace instances using an incremental or phased approach to an instance refresh\. This provides additional time for verification between sets of replacements\. If you choose not to enable checkpoints, the instances are replaced in one nearly continuous operation\.

   If you enable checkpoints, see [Enable checkpoints \(console\)](asg-adding-checkpoints-instance-refresh.md#enable-checkpoints-console) for additional steps\.

1. Enable or turn off **Skip matching**:
   + To skip replacing instances that already match your launch template, keep the **Enable skip matching** check box selected\.
   + If you turn off skip matching by clearing this check box, all instances can be replaced\.

   When you enable skip matching, instead of using your launch template, you can set a new launch template or a new version of the launch template\. You can do so in the **Desired configuration** section of the **Start instance refresh** page\. 
**Note**  
To use the skip matching feature to update an Auto Scaling group that currently uses a launch configuration, you must select a launch template in **Desired configuration**\. Using skip matching with a launch configuration is not supported\.

1. For **Standby instances**, choose **Ignore**, **Terminate**, or **Wait**\. This determines what happens if instances are found in `Standby` state\. For more information, see [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md)\.

   If you choose **Wait**, you must take additional steps to return these instances to service\. If you don't, the instance refresh replaces all `InService` instances and waits for one hour\. Then, if any `Standby` instances remain, the instance refresh fails\. To prevent this situation, choose to **Ignore** or **Terminate** these instances instead\. 

1. For **Scale\-in protected instances**, choose **Ignore**, **Replace**, or **Wait**\. This determines what happens if scale\-in protected instances are found\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

   If you choose **Wait**, you must take additional steps to remove scale\-in protection from these instances\. If you don't, the instance refresh replaces all unprotected instances and waits one hour\. Then, if any scale\-in protected instances remain, the instance refresh fails\. To prevent this situation, choose to **Ignore** or **Replace** these instances instead\. 

1. \(Optional\) Expand the **Desired configuration** section to specify updates that you want to make to your Auto Scaling group\.

   For this step, you can choose to use JSON or YAML syntax to edit parameter values instead of making selections in the console interface\. To do so, choose **Use code editor** instead of **Use console interface**\. The following procedure explains how to make selections using the console interface\.

   1. For **Update launch template**: 
      + If you *haven't* created a new launch template or a new launch template version for your Auto Scaling group, don't select this check box\.
      + If you've created a new launch template or a new launch template version, select this check box\. When you select this option, Amazon EC2 Auto Scaling shows you the current launch template and current launch template version\. It also lists any other available versions\. Choose the launch template and then choose the version\. 

        After you choose a version, you can see the version information\. This is the version of the launch template that will be used when replacing instances as part of an instance refresh\. If the instance refresh succeeds, this version of the launch template will also be used whenever new instances launch, such as when the group scales\.

   1. For **Choose a set of instance types and purchase options to override the instance type in the launch template**:
      + Don't select this check box if you want to use the instance type and purchase option you specified in your launch template\.
      + Select this check box if you want to override the instance type in the launch template or run Spot Instances\. You can either manually add each instance type or choose a primary instance type and recommendation option that retrieves any additional matching instance types for you\. If you plan to launch Spot Instances, we recommend adding a few different instance types\. This way, Amazon EC2 Auto Scaling can launch another instance type if there is insufficient instance capacity in your chosen Availability Zones\. For more information, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.
**Warning**  
Don't use Spot Instances with applications that can't handle a Spot Instance interruption\. Interruptions can occur if the Amazon EC2 Spot service needs to reclaim capacity\.

      If you select this check box, make sure that the launch template doesn't already request Spot Instances\. You can't use a launch template that requests Spot Instances to create an Auto Scaling group that uses multiple instance types and launches Spot and On\-Demand Instances\.
**Note**  
To configure these options on an Auto Scaling group that currently uses a launch configuration, you must select a launch template in **Update launch template**\. Overriding the instance type in your launch configuration is not supported\.

1. \(Optional\) For **Rollback settings**, choose **Enable auto rollback** to automatically roll back the instance refresh if it fails\.

   This setting cannot be enabled when the Auto Scaling group doesn't meet the prerequisites for using rollbacks\. 

   For more information, see [Undo changes with a rollback](instance-refresh-rollback.md)\.

1. Review all of your selections to confirm that everything is set up correctly\.

   At this point, it's a good idea to verify that the differences between the current and proposed changes won't affect your application in unexpected or unwanted ways\. To confirm that your instance type is compatible with your launch template, see [Instance type compatibility](asg-instance-refresh.md#instance-type-compatibility)\.

1. When you are satisfied with your instance refresh selections, choose **Start**\. 

### Start an instance refresh in the console \(mixed instances group\)<a name="starting-an-instance-refresh-in-the-console-mig"></a>

Use the following procedure if you have created an Auto Scaling group with a [mixed instances policy](ec2-auto-scaling-mixed-instances-groups.md)\. If you haven't defined a mixed instances policy for your group yet, see [Start an instance refresh in the console \(basic procedure\)](#starting-an-instance-refresh-in-the-console) to start an instance refresh\.

**To start an instance refresh**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up at the bottom of the **Auto Scaling groups** page\. 

1. On the **Instance refresh** tab, in **Active instance refresh**, choose **Start instance refresh**\.

1. For **Minimum healthy percentage**, enter the percentage of the Auto Scaling group that must remain healthy during an instance refresh\. Setting the minimum healthy percentage to 100 percent limits the rate of replacement to one instance at a time\. In contrast, setting it to 0 percent makes it replace all the instances at the same time\.

1. For **Instance warmup**, enter the number of seconds from when a new instance's state changes to `InService` to when it can receive traffic\. Amazon EC2 Auto Scaling waits this amount of time before moving on to replace the next instance\. 

   While warming up, a newly launched instance is also not counted toward the aggregated instance metrics of the Auto Scaling group \(such as `CPUUtilization`, `NetworkIn`, `NetworkOut`, and so on\)\. If you added scaling policies to the Auto Scaling group, the scaling activities run in parallel\. If you set a long interval for the instance refresh warm\-up period, it takes more time for newly launched instances to show up in the metrics\. Therefore, an adequate warm\-up period keeps Amazon EC2 Auto Scaling from scaling on stale metric data\.

   If you have already correctly defined a default instance warmup for the Auto Scaling group, then you don't need to change the instance warmup \(unless you want to override the default\)\. For more information about setting the default instance warmup, see [Set the default instance warmup for an Auto Scaling group](ec2-auto-scaling-default-instance-warmup.md)\.

1. \(Optional\) For **Checkpoints**, choose **Enable checkpoints** to replace instances using an incremental or phased approach to an instance refresh\. This provides additional time for verification between sets of replacements\. If you choose not to enable checkpoints, the instances are replaced in one nearly continuous operation\.

   If you enable checkpoints, see [Enable checkpoints \(console\)](asg-adding-checkpoints-instance-refresh.md#enable-checkpoints-console) for additional steps\.

1. Enable or turn off **Skip matching**:
   + To skip replacing instances that already match your launch template and any instance type overrides, keep the **Enable skip matching** check box selected\.
   + If you choose to turn off skip matching by clearing this check box, all instances can be replaced\.

   When you enable skip matching, instead of using your launch template, you can set a new launch template or a new version of the launch template\. You can do so in the **Desired configuration** section of the **Start instance refresh** page\. You can also update your instance type overrides in **Desired configuration**\. 

1. For **Standby instances**, choose **Ignore**, **Terminate**, or **Wait**\. This determines what happens if instances are found in `Standby` state\. For more information, see [Temporarily remove instances from your Auto Scaling group](as-enter-exit-standby.md)\.

   If you choose **Wait**, you must take additional steps to return these instances to service\. If you don't, the instance refresh replaces all `InService` instances and waits one hour\. Then, if any `Standby` instances remain, the instance refresh fails\. To prevent this situation, choose to **Ignore** or **Terminate** these instances instead\. 

1. For **Scale\-in protected instances**, choose **Ignore**, **Replace**, or **Wait**\. This determines what happens if scale\-in protected instances are found\. For more information, see [Use instance scale\-in protection](ec2-auto-scaling-instance-protection.md)\.

   If you choose **Wait**, you must take additional steps to remove scale\-in protection from these instances\. If you don't, the instance refresh replaces all unprotected instances and waits one hour\. Then, if any scale\-in protected instances remain, the instance refresh fails\. To prevent this situation, choose to **Ignore** or **Replace** these instances instead\.

1. In the **Desired configuration** section, do the following\. 

   For this step, you can choose to use JSON or YAML syntax to edit parameter values instead of making selections in the console interface\. To do so, choose **Use code editor** instead of **Use console interface**\. The following procedure explains how to make selections using the console interface\.

   1. For **Update launch template**: 
      + If you *haven't* created a new launch template or a new launch template version for your Auto Scaling group, don't select this check box\.
      + If you've created a new launch template or a new launch template version, select this check box\. When you select this option, Amazon EC2 Auto Scaling shows you the current launch template and current launch template version\. It also lists any other available versions\. Choose the launch template and then choose the version\. 

        After you choose a version, you can see the version information\. This is the version of the launch template that will be used when replacing instances as part of an instance refresh\. If the instance refresh succeeds, this version of the launch template will also be used whenever new instances launch, such as when the group scales\.

   1. For **Use these settings to override the instance type and purchase option defined in the launch template**: 

      By default, this check box is selected\. Amazon EC2 Auto Scaling populates each parameter with the value that's currently set in the *mixed instances policy* for the Auto Scaling group\. Only update values for the parameters that you want to change\. For guidance on these settings, see [Auto Scaling groups with multiple instance types and purchase options](ec2-auto-scaling-mixed-instances-groups.md)\.
**Warning**  
We recommend that you do not clear this check box\. Only clear it if you want to stop using a mixed instances policy\. After the instance refresh succeeds, Amazon EC2 Auto Scaling updates your group to match the **Desired configuration**\. If it no longer includes a mixed instances policy, Amazon EC2 Auto Scaling gradually terminates any Spot Instances that are currently running and replaces them with On\-Demand Instances\. Or, if your launch template requests Spot Instances, then Amazon EC2 Auto Scaling gradually terminates any On\-Demand Instances that are currently running and replaces them with Spot Instances\. 

1. \(Optional\) For **Rollback settings**, choose **Enable auto rollback** to automatically roll back the instance refresh if it fails for any reason\.

   This setting cannot be enabled when the Auto Scaling group doesn't meet the prerequisites for using rollbacks\. 

   For more information, see [Undo changes with a rollback](instance-refresh-rollback.md)\.

1. Review all of your selections to confirm that everything is set up correctly\.

   At this point, it's a good idea to verify that the differences between the current and proposed changes won't affect your application in unexpected or unwanted ways\. To confirm that your instance type is compatible with your launch template, see [Instance type compatibility](asg-instance-refresh.md#instance-type-compatibility)\.

   When you are satisfied with your instance refresh selections, choose **Start**\. 

## Start an instance refresh \(AWS CLI\)<a name="start-instance-refresh-cli"></a>

**To start an instance refresh**  
Use the following [start\-instance\-refresh](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/start-instance-refresh.html) command to start an instance refresh from the AWS CLI\. You can specify any preferences that you want to change in a JSON configuration file\. When you reference the configuration file, provide the file path and name as shown in the following example\.

```
aws autoscaling start-instance-refresh --cli-input-json file://config.json
```

Contents of `config.json`:

```
{
    "AutoScalingGroupName": "my-asg",
    "Preferences": {
      "InstanceWarmup": 60,
      "MinHealthyPercentage": 50,
      "AutoRollback": true,
      "ScaleInProtectedInstances": Ignore,
      "StandbyInstances": Terminate
    }
}
```

If preferences are not provided, default values are used\. For more information, see [Understand the default values for an instance refresh](understand-instance-refresh-default-values.md)\.

Example output:

```
{
    "InstanceRefreshId": "08b91cf7-8fa6-48af-b6a6-d227f40f1b9b"
}
```