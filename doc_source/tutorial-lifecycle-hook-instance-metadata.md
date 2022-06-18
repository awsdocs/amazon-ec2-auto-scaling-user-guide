# Tutorial: Configure user data to retrieve the target lifecycle state through instance metadata<a name="tutorial-lifecycle-hook-instance-metadata"></a>

A common way to create custom actions for lifecycle hooks is to use notifications that Amazon EC2 Auto Scaling sends to other services, such as Amazon EventBridge\. However, you can avoid having to create additional infrastructure by instead using a user data script to move the code that configures instances and completes the lifecycle action into the instances themselves\. 

The following tutorial shows you how to get started using a user data script and instance metadata\. You create a basic Auto Scaling group configuration with a user data script that reads the [target lifecycle state](retrieving-target-lifecycle-state-through-imds.md) of the instances in your group and performs a callback action at a specific phase of an instance's lifecycle to continue the launch process\.

**Topics**
+ [Step 1: Create an IAM role with permissions to complete lifecycle actions](#instance-metadata-create-iam-role)
+ [Step 2: Create a launch template and include the IAM role and a user data script](#instance-metadata-create-hello-world-function)
+ [Step 3: Create an Auto Scaling group](#instance-metadata-create-auto-scaling-group)
+ [Step 4: Add a lifecycle hook](#instance-metadata-add-lifecycle-hook)
+ [Step 5: Test and verify the functionality](#instance-metadata-testing-hook)
+ [Step 6: Clean up](#instance-metadata-lifecycle-hooks-tutorial-cleanup)

## Step 1: Create an IAM role with permissions to complete lifecycle actions<a name="instance-metadata-create-iam-role"></a>

When you use the AWS CLI or an AWS SDK to send a callback to complete lifecycle actions, you must use an IAM role with permissions to complete lifecycle actions\. 

**To create the policy**

1. Open the [Policies page](https://console.aws.amazon.com/iam/home?#/policies) of the IAM console, and then choose **Create policy**\.

1. Choose the **JSON** tab\.

1. In the **Policy Document** box, copy and paste the following policy document into the box\. Replace the **sample text** with your account number and the name of the Auto Scaling group that you want to create \(**TestAutoScalingEvent\-group**\)\.

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "autoscaling:CompleteLifecycleAction"
         ],
         "Resource": "arn:aws:autoscaling:*:123456789012:autoScalingGroup:*:autoScalingGroupName/TestAutoScalingEvent-group"
       }
     ]
   }
   ```

1. Choose **Next:Tags**, and then **Next:Review**\. 

1. For **Name**, enter **TestAutoScalingEvent\-policy**\. Choose **Create policy**\.

When you finish creating the policy, you can create a role that uses it\.

**To create the role**

1. In the navigation pane on the left, choose **Roles**\.

1. Choose **Create role**\.

1. For **Select trusted entity**, choose **AWS service**\.

1. For your use case, choose **EC2** and then choose **Next**\. 

1. Under **Add permissions**, choose the policy that you created \(**TestAutoScalingEvent\-policy**\)\. Then, choose **Next**\. 

1. On the **Name, review, and create** page, for **Role name**, enter **TestAutoScalingEvent\-role** and choose **Create role**\. 

## Step 2: Create a launch template and include the IAM role and a user data script<a name="instance-metadata-create-hello-world-function"></a>

Create a launch template to use with your Auto Scaling group\. Include the IAM role you created and the provided sample user data script\.

**To create a launch template**

1. Open the [Launch templates page](https://console.aws.amazon.com/ec2/v2/#LaunchTemplates) on the Amazon EC2 console\.

1. Choose **Create launch template**\.

1. For **Launch template name**, enter **TestAutoScalingEvent\-template**\.

1. Under **Auto Scaling guidance**, select the check box\. 

1. For **Application and OS Images \(Amazon Machine Image\)**, choose Amazon Linux 2 \(HVM\), SSD Volume Type, 64\-bit \(x86\) from the **Quick Start** list\. 

1. For **Instance type**, choose a type of Amazon EC2 instance \(for example, "t2\.micro"\)\.

1. For **Advanced details**, expand the section to view the fields\. 

1. For **IAM instance profile**, choose the IAM instance profile name of your IAM role\. An instance profile is a container for an IAM role that allows Amazon EC2 to pass the IAM role to an instance when the instance is launched\.

   When you used the IAM console to create an IAM role, the console automatically created an instance profile with the same name as its corresponding role\.

1. For **User data**, copy and paste the following sample user data script into the field\. Replace the sample text for `group_name` with the name of the Auto Scaling group that you want to create and `region` with the AWS Region you want your Auto Scaling group to use\.

   ```
   #!/bin/bash
   
   function get_target_state {
       echo $(curl -s http://169.254.169.254/latest/meta-data/autoscaling/target-lifecycle-state)
   }
   
   function get_instance_id {
       echo $(curl -s http://169.254.169.254/latest/meta-data/instance-id)
   }
   
   function complete_lifecycle_action {
       instance_id=$(get_instance_id)
       group_name='TestAutoScalingEvent-group'
       region='us-west-2'
    
       echo $instance_id
       echo $region
       echo $(aws autoscaling complete-lifecycle-action \
         --lifecycle-hook-name TestAutoScalingEvent-hook \
         --auto-scaling-group-name $group_name \
         --lifecycle-action-result CONTINUE \
         --instance-id $instance_id \
         --region $region)
   }
   
   function main {
       while true
       do
           target_state=$(get_target_state)
           if [ \"$target_state\" = \"InService\" ]; then
               # Change hostname
               export new_hostname="${group_name}-$instance_id"
               hostname $new_hostname
               # Send callback
               complete_lifecycle_action
           fi
           echo $target_state
           sleep 5
       done
   }
   
   main
   ```

   This simple user data script does the following:
   + Calls the instance metadata to retrieve the target lifecycle state and instance ID from the instance metadata
   + Retrieves the target lifecycle state repeatedly until it changes to `InService`
   + Changes the hostname of the instance to the instance ID prepended with the name of the Auto Scaling group, if the target lifecycle state is `InService`
   + Sends a callback by calling the complete\-lifecycle\-action CLI command to signal Amazon EC2 Auto Scaling to `CONTINUE` the EC2 launch process

1. Choose **Create launch template**\.

1. On the confirmation page, choose **Create Auto Scaling group**\.

## Step 3: Create an Auto Scaling group<a name="instance-metadata-create-auto-scaling-group"></a>

After you create your launch template, create an Auto Scaling group\.

**To create an Auto Scaling group**

1. On the **Choose launch template or configuration** page, for **Auto Scaling group name**, enter a name for your Auto Scaling group \(**TestAutoScalingEvent\-group**\)\.

1. Choose **Next** to go to the **Choose instance launch options** page\. 

1. For **Network**, choose a VPC\.

1. For **Availability Zones and subnets**, choose one or more subnets from one or more Availability Zones\.

1. In the **Instance type requirements** section, use the default setting to simplify this step\. \(Do not override the launch template\.\) For this tutorial, you will launch only one On\-Demand Instance using the instance type specified in your launch template\. 

1. Choose **Next**, and then choose **Skip to review** at the bottom of the next screen\. 

1. On the **Review** page, review the details of your Auto Scaling group, and then choose **Create Auto Scaling group**\.

## Step 4: Add a lifecycle hook<a name="instance-metadata-add-lifecycle-hook"></a>

Add a lifecycle hook to hold the instance in a wait state until your lifecycle action is complete\.

**To add a lifecycle hook**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2/v2/home?#AutoScalingGroups) of the Amazon EC2 console\.

1. Select the check box next to your Auto Scaling group\.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page\. 

1. In the lower pane, on the **Instance management** tab, in **Lifecycle hooks**, choose **Create lifecycle hook**\.

1. To define a lifecycle hook, do the following:

   1. For **Lifecycle hook name**, enter **TestAutoScalingEvent\-hook**\.

   1. For **Lifecycle transition**, choose **Instance launch**\.

   1. For **Heartbeat timeout**, enter **300** for the number of seconds to wait for a callback from your user data script\.

   1. For **Default result**, choose **ABANDON**\. If the hook times out without receiving a callback from your user data script, the Auto Scaling group terminates the new instance\.

   1. \(Optional\) Keep **Notification metadata** blank\.

1. Choose **Create**\.

## Step 5: Test and verify the functionality<a name="instance-metadata-testing-hook"></a>

To test the functionality, update the Auto Scaling group by increasing the desired capacity of the Auto Scaling group by 1\. The user data script runs and starts to check the instance's target lifecycle state soon after the instance launches\. The script changes the hostname and sends a callback action when the target lifecycle state is `InService`\. This usually takes only a few seconds to finish\.

**To increase the size of the Auto Scaling group**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2/v2/home?#AutoScalingGroups) of the Amazon EC2 console\.

1. Select the check box next to your Auto Scaling group\. View details in a lower pane while still seeing the top rows of the upper pane\. 

1. In the lower pane, on the **Details** tab, choose **Group details**, **Edit**\.

1. For **Desired capacity**, increase the current value by 1\.

1. Choose **Update**\. While the instance is being launched, the **Status** column in the upper pane displays a status of *Updating capacity*\. 

After increasing the desired capacity, you can verify that your instance has successfully launched and is not terminated from the description of scaling activities\. 

**To view the scaling activity**

1. Return to the **Auto Scaling groups** page and select your group\.

1. On the **Activity** tab, under **Activity history**, the **Status** column shows whether your Auto Scaling group has successfully launched an instance\. 

1. If the user data script fails, after the timeout period passes, you see a scaling activity with a status of `Canceled` and a status message of `Instance failed to complete user's Lifecycle Action: Lifecycle Action with token e85eb647-4fe0-4909-b341-a6c42EXAMPLE was abandoned: Lifecycle Action Completed with ABANDON Result`\.

## Step 6: Clean up<a name="instance-metadata-lifecycle-hooks-tutorial-cleanup"></a>

If you are done working with the resources that you created for this tutorial, use the following steps to delete them\.

**To delete the lifecycle hook**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2/v2/home?#AutoScalingGroups) of the Amazon EC2 console\.

1. Select the check box next to your Auto Scaling group\.

1. On the **Instance management** tab, in **Lifecycle hooks**, choose the lifecycle hook \(`TestAutoScalingEvent-hook`\)\.

1. Choose **Actions**, **Delete**\.

1. Choose **Delete** again to confirm\.

**To delete the launch template**

1. Open the [Launch templates page](https://console.aws.amazon.com/ec2/v2/#LaunchTemplates) on the Amazon EC2 console\.

1. Select your launch template \(`TestAutoScalingEvent-template`\) and then choose **Actions**, **Delete template**\.

1. When prompted for confirmation, choose **Delete launch template**\.

If you are done working with the example Auto Scaling group, delete it\. You can also delete the IAM role and permissions policy that you created\.

**To delete the Auto Scaling group**

1. Open the [Auto Scaling groups page](https://console.aws.amazon.com/ec2/v2/home?#AutoScalingGroups) of the Amazon EC2 console\.

1. Select the check box next to your Auto Scaling group \(`TestAutoScalingEvent-group`\) and choose **Delete**\. 

1. When prompted for confirmation, choose **Delete**\.

   A loading icon in the **Name** column indicates that the Auto Scaling group is being deleted\. It takes a few minutes to terminate the instances and delete the group\. 

**To delete the IAM role**

1. Open the [Roles page](https://console.aws.amazon.com/iam/home?#/roles) of the IAM console\.

1. Select the function's role \(`TestAutoScalingEvent-role`\)\.

1. Choose **Delete role**\.

1. In the **Delete role** dialog box, choose **Yes, delete**\.

**To delete the IAM policy**

1. Open the [Policies page](https://console.aws.amazon.com/iam/home?#/policies) of the IAM console\.

1. Select the policy that you created \(`TestAutoScalingEvent-policy`\)\.

1. Choose **Delete policy**\.

1. In the **Delete policy** dialog box, choose **Yes, delete**\.