# Examples for creating and managing launch templates with the AWS Command Line Interface \(AWS CLI\)<a name="examples-launch-templates-aws-cli"></a>

You can create and manage launch templates through the AWS Management Console, AWS CLI, or SDKs\. This section shows you examples of creating and managing launch templates for Amazon EC2 Auto Scaling from the AWS CLI\.

**Topics**
+ [Example usage](#example-usage)
+ [Creating a basic launch template](#example-simple-launch-template)
+ [Specifying tags that tag instances at launch](#example-tag-instances)
+ [Specifying an IAM role to pass to instances](#example-iam-profile)
+ [Assigning public IP addresses](#example-primary-network-interface)
+ [Specifying a user data script that configures instances at launch](#example-user-data)
+ [Specifying a block device mapping](#example-block-device-mapping)
+ [Specifying Dedicated Hosts to bring software licenses from external vendors](#example-dedicated-hosts)
+ [Specifying an existing network interface](#example-existing-eni-launch-template)
+ [Creating multiple network interfaces](#example-multiple-efa-enabled-network-interfaces)
+ [Managing your launch templates](#launch-templates-additional-cli-commands)
+ [Updating an Auto Scaling group to use a launch template](#update-asg-launch-template-cli)

## Example usage<a name="example-usage"></a>

```
{
    "LaunchTemplateName": "my-template-for-auto-scaling",
    "VersionDescription": "test description",
    "LaunchTemplateData": {
        "ImageId": "ami-04d5cc9b88example",
        "InstanceType": "t2.micro",
        "SecurityGroupIds": [
            "sg-903004f88example"
        ], 
        "KeyName": "MyKeyPair",
        "Monitoring": {
            "Enabled": true
        },
        "Placement": {
            "Tenancy": "dedicated"
        },
        "CreditSpecification": {
            "CpuCredits": "unlimited"
        },
        "MetadataOptions": {
            "HttpTokens": "required",
            "HttpPutResponseHopLimit": 1,
            "HttpEndpoint": "enabled"
        }
    }
}
```

## Creating a basic launch template<a name="example-simple-launch-template"></a>

To create a basic launch template, use the [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command as follows, with these modifications: 
+ Replace `ami-04d5cc9b88example` with the ID of the AMI from which to launch the instances\.
+ Replace `t2.micro` with an instance type that is compatible with the AMI that you specified\.

This example creates a launch template with the name *my\-template\-for\-auto\-scaling*\. If the instances created by this launch template are launched in a default VPC, they receive a public IP address by default\. If the instances are launched in a nondefault VPC, they do not receive a public IP address by default\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

For more information about quoting JSON\-formatted parameters, see [Using quotation marks with strings in the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-parameters-quoting-strings.html) in the *AWS Command Line Interface User Guide*\.

Alternatively, you can specify the JSON\-formatted parameters in a configuration file\.

The following example creates a basic launch template, referencing a configuration file for launch template parameter values\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data file://config.json
```

Contents of `config.json`:

```
{ 
    "ImageId":"ami-04d5cc9b88example",
    "InstanceType":"t2.micro"
}
```

## Specifying tags that tag instances at launch<a name="example-tag-instances"></a>

The following example adds a tag \(for example, `purpose=webserver`\) to instances at launch\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"TagSpecifications":[{"ResourceType":"instance","Tags":[{"Key":"purpose","Value":"webserver"}]}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

**Note**  
If instance tags are specified both in the launch template and in the Auto Scaling group configuration, all the tags are merged\. If there is a collision on the tag's key, then the value in the Auto Scaling group configuration takes precedence\. 

## Specifying an IAM role to pass to instances<a name="example-iam-profile"></a>

The following example specifies the name of the instance profile associated with the IAM role to pass to instances at launch\. For more information, see [IAM role for applications that run on Amazon EC2 instances](us-iam-role.md)\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
--launch-template-data '{"IamInstanceProfile":{"Name":"my-instance-profile"},"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Assigning public IP addresses<a name="example-primary-network-interface"></a>

The following [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) example configures the launch template to assign public addresses to instances launched in a nondefault VPC\. 

**Note**  
When you specify a network interface, specify a value for `Groups` that corresponds to security groups for the VPC that your Auto Scaling group will launch instances into\. Specify the VPC subnets as properties of the Auto Scaling group\. 



```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"AssociatePublicIpAddress":true,"Groups":["sg-903004f88example"],"DeleteOnTermination":true}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Specifying a user data script that configures instances at launch<a name="example-user-data"></a>

The following example specifies a user data script as a base64\-encoded string that configures instances at launch\. The [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command requires base64\-encoded user data\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
--launch-template-data '{"UserData":"IyEvYmluL2Jhc...","ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Specifying a block device mapping<a name="example-block-device-mapping"></a>

The following [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) example creates a launch template with a block device mapping: a 22\-gigabyte EBS volume mapped to `/dev/xvdcz`\. The `/dev/xvdcz` volume uses the General Purpose SSD \(gp2\) volume type and is deleted when terminating the instance it is attached to\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"BlockDeviceMappings":[{"DeviceName":"/dev/xvdcz","Ebs":{"VolumeSize":22,"VolumeType":"gp2","DeleteOnTermination":true}}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Specifying Dedicated Hosts to bring software licenses from external vendors<a name="example-dedicated-hosts"></a>

If you specify *host* tenancy, you can specify a host resource group and a License Manager license configuration to bring eligible software licenses from external vendors\. Then, you can use the licenses on EC2 instances by using the following [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"Placement":{"Tenancy":"host","HostResourceGroupArn":"arn"},"LicenseSpecifications":[{"LicenseConfigurationArn":"arn"}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Specifying an existing network interface<a name="example-existing-eni-launch-template"></a>

The following [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) example configures the primary network interface to use an existing network interface\.

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"NetworkInterfaceId":"eni-b9a5ac93","DeleteOnTermination":false}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

## Creating multiple network interfaces<a name="example-multiple-efa-enabled-network-interfaces"></a>

The following [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) example adds a secondary network interface\. The primary network interface has a device index of 0, and the secondary network interface has a device index of 1\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"Groups":["sg-903004f88example"],"DeleteOnTermination":true},{"DeviceIndex":1,"Groups":["sg-903004f88example"],"DeleteOnTermination":true}],"ImageId":"ami-04d5cc9b88example","InstanceType":"t2.micro"}'
```

If you use an instance type that supports multiple network cards and Elastic Fabric Adapters \(EFAs\), you can add a secondary interface to a secondary network card and enable EFA by using the following [create\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html) command\. For more information, see [Adding an EFA to a launch template](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa-working-with.html#efa-launch-template) in the *Amazon EC2 User Guide for Linux Instances*\. 

```
aws ec2 create-launch-template --launch-template-name my-template-for-auto-scaling --version-description version1 \
  --launch-template-data '{"NetworkInterfaces":[{"NetworkCardIndex":0,"DeviceIndex":0,"Groups":["sg-7c2270198example"],"InterfaceType":"efa","DeleteOnTermination":true},{"NetworkCardIndex":1,"DeviceIndex":1,"Groups":["sg-7c2270198example"],"InterfaceType":"efa","DeleteOnTermination":true}],"ImageId":"ami-09d95fab7fexample","InstanceType":"p4d.24xlarge"}'
```

**Warning**  
The p4d\.24xlarge instance type incurs higher costs than the other examples in this section\. For more information about pricing for P4d instances, see [Amazon EC2 P4d Instances pricing](http://aws.amazon.com/ec2/instance-types/p4/)\.

**Note**  
Attaching multiple network interfaces from the same subnet to an instance can introduce asymmetric routing, especially on instances using a variant of non\-Amazon Linux\. If you need this type of configuration, you must configure the secondary network interface within the OS\. For an example, see [How can I make my secondary network interface work in my Ubuntu EC2 instance?](http://aws.amazon.com/premiumsupport/knowledge-center/ec2-ubuntu-secondary-network-interface/) in the AWS Knowledge Center\.

## Managing your launch templates<a name="launch-templates-additional-cli-commands"></a>

The AWS CLI includes several other commands that help you manage your launch templates\. 

**Topics**
+ [Listing and describing your launch templates](#describe-launch-template-aws-cli)
+ [Creating a launch template version](#example-create-launch-template-version)
+ [Deleting a launch template version](#example-delete-launch-template-version)
+ [Deleting a launch template](#example-delete-launch-template)

### Listing and describing your launch templates<a name="describe-launch-template-aws-cli"></a>

You can use two AWS CLI commands to get information about your launch templates: [describe\-launch\-templates](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html) and [describe\-launch\-template\-versions](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html)\. 

The [describe\-launch\-templates](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-templates.html) command enables you to get a list of any of the launch templates that you have created\. You can use an option to filter results on a launch template name, create time, tag key, or tag key\-value combination\. This command returns summary information about any of your launch templates, including the launch template identifier, latest version, and default version\. 

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

The following [describe\-launch\-template\-versions](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-launch-template-versions.html) command provides information describing the versions of the specified launch template *my\-template\-for\-auto\-scaling*\.

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

The following [create\-launch\-template\-version](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template-version.html) command creates a new launch template version based on version 1 of the launch template and specifies a different AMI ID\.

```
aws ec2 create-launch-template-version --launch-template-id lt-068f72b729example --version-description version2 \
  --source-version 1 --launch-template-data "ImageId=ami-c998b6b2example"
```

To set the default version of the launch template, use the [modify\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-launch-template.html) command\. 

### Deleting a launch template version<a name="example-delete-launch-template-version"></a>

The following [delete\-launch\-template\-versions](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template-versions.html) command deletes the specified launch template version\.

```
aws ec2 delete-launch-template-versions --launch-template-id lt-068f72b729example --versions 1
```

### Deleting a launch template<a name="example-delete-launch-template"></a>

If you no longer require a launch template, you can delete it using the following [delete\-launch\-template](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-launch-template.html) command\. Deleting a launch template deletes all of its versions\. 

```
aws ec2 delete-launch-template --launch-template-id lt-068f72b729example
```

## Updating an Auto Scaling group to use a launch template<a name="update-asg-launch-template-cli"></a>

You can use the [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to add a launch template to an existing Auto Scaling group\.

**Note**  
If you switch your Auto Scaling group from using a launch configuration, be sure that your permissions are up to date\. In order to use a launch template, you need specific [permissions](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html#launch-templates-permissions)\.

### Updating an Auto Scaling group to use the latest version of a launch template<a name="example-update-asg-launch-template-latest-version"></a>

The following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command updates the specified Auto Scaling group to use the latest version of the specified launch template\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateId=lt-068f72b729example,Version='$Latest'
```

### Updating an Auto Scaling group to use a specific version of a launch template<a name="example-update-asg-launch-template-specific-version"></a>

The following [update\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command updates the specified Auto Scaling group to use a specific version of the specified launch template\.

```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template-for-auto-scaling,Version='2'
```