# Configure instance tenancy with a launch configuration<a name="auto-scaling-dedicated-instances"></a>

Tenancy defines how EC2 instances are distributed across physical hardware and affects pricing\. There are three tenancy options available: 
+ Shared \(`default`\) — Multiple AWS accounts may share the same physical hardware\. 
+ Dedicated Instance \(`dedicated`\) — Your instance runs on single\-tenant hardware\. 
+ Dedicated Host \(`host`\) — Your instance runs on a physical server with EC2 instance capacity fully dedicated to your use, an isolated server with configurations that you can control\. 

This topic describes how to launch Dedicated Instances in your Auto Scaling group by specifying settings in a launch configuration\. For pricing information and to learn more about Dedicated Instances, see the [Amazon EC2 dedicated instances](https://aws.amazon.com/ec2/purchasing-options/dedicated-instances/) product page and [Dedicated Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html) in the *Amazon EC2 User Guide for Linux Instances*\. 

You can configure tenancy for EC2 instances using a launch configuration or launch template\. However, the `host` tenancy value cannot be used with a launch configuration\. Use the `default` or `dedicated` tenancy values only\.

**Important**  
To use a tenancy value of `host`, you must use a launch template\. For more information, see [Create a launch template for an Auto Scaling group](create-launch-template.md)\. Before launching Dedicated Hosts, we recommend that you become familiar with launching and managing Dedicated Hosts using [AWS License Manager](http://aws.amazon.com/ec2/dedicated-hosts/)\. For more information, see the [License Manager User Guide](https://docs.aws.amazon.com/license-manager/latest/userguide/)\.

When you create a VPC, by default its tenancy attribute is set to `default`\. In such a VPC, you can launch instances with a tenancy value of `dedicated` so that they run as single\-tenancy instances\. Otherwise, they run as shared\-tenancy instances by default\. If you set the tenancy attribute of a VPC to `dedicated`, all instances launched in the VPC run as single\-tenancy instances\. 

When you create a launch configuration, the default value for the instance placement tenancy is `null` and the instance tenancy is controlled by the tenancy attribute of the VPC\. You can specify the instance placement tenancy for your launch configuration as `default` or `dedicated` using the [create\-launch\-configuration](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) CLI command with the `--placement-tenancy` option\. 

The following table summarizes the instance placement tenancy of the Auto Scaling instances launched in a VPC\.


| Launch configuration tenancy | VPC tenancy = `default` | VPC tenancy = `dedicated` | 
| --- | --- | --- | 
|  not specified  |  shared\-tenancy instances  |  Dedicated Instances  | 
|   `default`   |  shared\-tenancy instances  |  Dedicated Instances  | 
|   `dedicated`   |  Dedicated Instances  |  Dedicated Instances  | 

**To create a launch configuration that creates Dedicated Instances \(AWS CLI\)**  
Use the following [create\-launch\-configuration](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-launch-configuration.html) command to create a launch configuration that sets the launch configuration tenancy to `dedicated`\.

```
aws autoscaling create-launch-configuration --launch-configuration-name my-launch-config --placement-tenancy dedicated --image-id ...
```

You can use the following [describe\-launch\-configurations](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to verify the instance placement tenancy of the launch configuration\.

```
aws autoscaling describe-launch-configurations --launch-configuration-names my-launch-config
```

The following is example output for a launch configuration that creates Dedicated Instances\. The `PlacementTenancy` parameter is only part of the output for this command when you explicitly set the instance placement tenancy\.

```
{
    "LaunchConfigurations": [
        {
            "UserData": null,
            "EbsOptimized": false,
            "PlacementTenancy": "dedicated",
            "LaunchConfigurationARN": "arn",
            "InstanceMonitoring": {
                "Enabled": true
            },
            "ImageId": "ami-b5a7ea85",
            "CreatedTime": "2020-03-08T23:39:49.011Z",
            "BlockDeviceMappings": [],
            "KeyName": null,
            "SecurityGroups": [],
            "LaunchConfigurationName": "my-launch-config",
            "KernelId": null,
            "RamdiskId": null,
            "InstanceType": "m3.medium"
        }
    ]
}
```