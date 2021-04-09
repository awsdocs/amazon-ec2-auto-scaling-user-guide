# Examples for creating and managing launch templates with the AWS Command Line Interface \(AWS CLI\)<a name="examples-launch-templates-aws-cli"></a>

You can create and manage launch templates through the AWS Management Console, AWS CLI, or SDKs\. This section shows you examples of creating and managing launch templates for Amazon EC2 Auto Scaling from the AWS CLI\.

**Topics**
+ [Creating a simple launch template](#example-simple-launch-template)
+ [Specifying tags that tag instances on creation](#example-tag-instances)
+ [Specifying an existing network interface](#example-existing-eni-launch-template)
+ [Assigning public IP addresses](#example-primary-network-interface)
+ [Adding a secondary network interface](#example-secondary-network-interface)
+ [Creating multiple EFA\-enabled network interfaces](#example-multiple-efa-enabled-network-interfaces)
+ [Specifying a block device mapping](#example-block-device-mapping)
+ [Managing your launch templates](#launch-templates-additional-cli-commands)
+ [Updating an Auto Scaling group to use a launch template](#update-asg-launch-template-cli)

## Creating a simple launch template<a name="example-simple-launch-template"></a>

To create a simple launch template, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command as follows, with these modifications: 
+ Replace `ami-04d5cc9b88example` with the ID of the AMI from which to launch the instances\.
+ Replace `t2.micro` with an instance type that is compatible with the AMI that you specified\.
+ Replace `sg-903004f88example` with a security group for the VPC that your Auto Scaling group will launch instances into\. 

This example creates a launch template with the name *my\-template\-for\-auto\-scaling*\. If the instances created by this launch template are launched in a default VPC, they receive a public IP address by default\. If the instances are launched in a nondefault VPC, they do not receive a public IP address by default\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro","SecurityGroupIds":["sg-903004f88example"]}'
```

For more information about quoting JSON\-formatted parameters, see [Using quotation marks with strings in the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-parameters-quoting-strings.html) in the *AWS Command Line Interface User Guide*\.

Alternatively, you can specify the JSON\-formatted parameters in a configuration file\.

The following example creates a simple launch template, referencing a configuration file for launch template parameter values\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data file://config.json
```

Contents of `config.json`:

```
{ 
    "ImageId":"ami-04d5cc9b88example",
    "InstanceType":"t2.micro",
    "SecurityGroupIds":["sg-903004f88example"]
}
```

## Specifying tags that tag instances on creation<a name="example-tag-instances"></a>

The following example adds a tag \(for example, `purpose=webserver`\) to instances at launch\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro","SecurityGroupIds":["sg-903004f88example"],"TagSpecifications":[{"ResourceType":"instance","Tags":[{"Key":"purpose","Value":"webserver"}]}]}'
```

## Specifying an existing network interface<a name="example-existing-eni-launch-template"></a>

The following example configures the primary network interface to use an existing network interface\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"NetworkInterfaceId":"eni-b9a5ac93","DeleteOnTermination":false}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Assigning public IP addresses<a name="example-primary-network-interface"></a>

The following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) example configures the launch template to assign public addresses to instances launched in a nondefault VPC\. Note that when you specify a network interface, specify a value for `Groups` that corresponds to security groups for the VPC that your Auto Scaling group will launch instances into, but specify the VPC subnets as properties of the Auto Scaling group\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"AssociatePublicIpAddress":true,"Groups":["sg-903004f88example"],"DeleteOnTermination":true}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Adding a secondary network interface<a name="example-secondary-network-interface"></a>

The following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) example adds a secondary network interface\. The primary network interface has a device index of 0, and the secondary network interface has a device index of 1\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"Groups":["sg-903004f88example"],"DeleteOnTermination":true},{"DeviceIndex":1,"Groups":["sg-903004f88example"],"DeleteOnTermination":true}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

**Note**  
Attaching multiple network interfaces from the same subnet to an instance can introduce asymmetric routing, especially on instances using a variant of non\-Amazon Linux\. If you need this type of configuration, you must configure the secondary network interface within the OS\. For an example, see [How can I make my secondary network interface work in my Ubuntu EC2 instance?](http://aws.amazon.com/premiumsupport/knowledge-center/ec2-ubuntu-secondary-network-interface/) in the AWS Knowledge Center\.

## Creating multiple EFA\-enabled network interfaces<a name="example-multiple-efa-enabled-network-interfaces"></a>

If you use an instance type that supports multiple network cards and Elastic Fabric Adapters \(EFAs\), you can add a secondary interface to a secondary network card and enable EFA using the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command\. For more information, see [Adding an EFA to a launch template](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa-working-with.html#efa-launch-template) in the *Amazon EC2 User Guide for Linux Instances*\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"ImageId":"ami-09d95fab7fexample","InstanceType":"p4d.24xlarge","NetworkInterfaces":[{"NetworkCardIndex":0,"DeviceIndex":0,"Groups":["sg-7c2270198example"],"InterfaceType":"efa","DeleteOnTermination":true},{"NetworkCardIndex":1,"DeviceIndex":1,"Groups":["sg-7c2270198example"],"InterfaceType":"efa","DeleteOnTermination":true}]}'
```

**Warning**  
The p4d\.24xlarge instance type incurs higher costs than the other examples in this section\. For more information about pricing for P4d instances, see [Amazon EC2 P4d Instances pricing](http://aws.amazon.com/ec2/instance-types/p4/)\.

## Specifying a block device mapping<a name="example-block-device-mapping"></a>

The following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) example creates a launch template with a block device mapping: a 22 gigabyte EBS volume mapped to /dev/xvdcz\. The /dev/xvdcz volume uses the General Purpose SSD \(gp2\) volume type and is deleted when terminating the instance it is attached to\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro","SecurityGroupIds":["sg-903004f88example"],"BlockDeviceMappings":[{"DeviceName":"/dev/xvdcz","Ebs":{"VolumeSize":22,"VolumeType":"gp2","DeleteOnTermination":true}}]}'
```

## Managing your launch templates<a name="launch-templates-additional-cli-commands"></a>

The AWS CLI includes several other commands that help you manage your launch templates\. 

**Topics**
+ [Listing and describing your launch templates](#describe-launch-template-aws-cli)
+ [Creating a launch template version](#example-create-launch-template-version)
+ [Deleting a launch template version](#example-delete-launch-template-version)
+ [Deleting a launch template](#example-delete-launch-template)

### Listing and describing your launch templates<a name="describe-launch-template-aws-cli"></a>

You can use two AWS CLI commands to get information about your launch templates: [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html) and [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html)\. 

The [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html) command enables you to get a list of any of the launch templates that you have created\. You can use an option to filter results on a launch template name, create time, tag key, or tag key/value combination\. This command returns summary information about any of your launch templates, including the launch template identifier, latest version, and default version\. 

The following example provides a summary of the specified launch template\.

```
aws ec2 describe-launch-templates --launch-template-names my-template-for-auto-scaling
```

The following is an example response\.

```
{
    "LaunchTemplates": [
        {
            "LaunchTemplateId": "lt-068f72b729example",
            "LaunchTemplateName": "my-template-for-auto-scaling",
            "CreateTime": "2020-02-28T19:52:27.000Z",
            "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
            "DefaultVersionNumber": 1,
            "LatestVersionNumber": 1
        }
    ]
}
```

If you don't use the `--launch-template-names` option to limit the output to one launch template, information on all of your launch templates is returned\.

The following [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html) command provides information describing the versions of the specified launch template *my\-template\-for\-auto\-scaling*\.

```
aws ec2 describe-launch-template-versions --launch-template-id lt-068f72b729example
```

The following is an example response\.

```
{
    "LaunchTemplateVersions": [
        {
            "VersionDescription": "version1",
            "LaunchTemplateId": "lt-068f72b729example",
            "LaunchTemplateName": "my-template-for-auto-scaling",
            "VersionNumber": 1,
            "CreatedBy": "arn:aws:iam::123456789012:user/Bob",
            "LaunchTemplateData": {
                "TagSpecifications": [
                    {
                        "ResourceType": "instance",
                        "Tags": [
                            {
                                "Key": "purpose",
                                "Value": "webserver"
                            }
                        ]
                    }
                ],
                "ImageId": "ami-04d5cc9b88example",
                "InstanceType": "t2.micro",
                "NetworkInterfaces": [
                    {
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "Groups": [
                            "sg-903004f88example"
                        ],
                        "AssociatePublicIpAddress": true
                    }
                ]
            },
            "DefaultVersion": true,
            "CreateTime": "2020-02-28T19:52:27.000Z"
        }
    ]
}
```

### Creating a launch template version<a name="example-create-launch-template-version"></a>

The following [https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template-version.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template-version.html) command creates a new launch template version based on version 1 of the launch template and specifies a different AMI ID\.

```
aws ec2 create-launch-template-version --launch-template-id lt-068f72b729example --version-description version2 \
  --source-version 1 --launch-template-data '{"ImageId":"ami-c998b6b2example"}'
```

To set the default version of the launch template, use the [https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-launch-template.html) command\. 

### Deleting a launch template version<a name="example-delete-launch-template-version"></a>

The following [https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template-versions.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template-versions.html) command deletes the specified launch template version\.

```
aws ec2 delete-launch-template-versions --launch-template-id lt-068f72b729example --versions 1
```

### Deleting a launch template<a name="example-delete-launch-template"></a>

If you no longer require a launch template, you can delete it using the following [https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template.html) command\. Deleting a launch template deletes all of its versions\. 

```
aws ec2 delete-launch-template --launch-template-id lt-068f72b729example
```

## Updating an Auto Scaling group to use a launch template<a name="update-asg-launch-template-cli"></a>

You can use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to add a launch template to an existing Auto Scaling group\.

**Note**  
If you switch your Auto Scaling group from using a launch configuration, be sure that your permissions are up\-to\-date\. In order to use a launch template, you need specific permissions\. For more information, see [Launch template support](ec2-auto-scaling-launch-template-permissions.md)\.

### Updating an Auto Scaling group to use the latest version of a launch template<a name="example-update-asg-launch-template-latest-version"></a>

The following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command updates the specified Auto Scaling group to use the latest version of the specified launch template\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateId=lt-068f72b729example,Version='$Latest'
```

### Updating an Auto Scaling group to use a specific version of a launch template<a name="example-update-asg-launch-template-specific-version"></a>

The following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command updates the specified Auto Scaling group to use a specific version of the specified launch template\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template-for-auto-scaling,Version='2'
```