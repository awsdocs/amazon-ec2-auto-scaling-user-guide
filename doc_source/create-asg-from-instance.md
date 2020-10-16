# Creating an Auto Scaling group using an EC2 instance<a name="create-asg-from-instance"></a>

Creating an Auto Scaling group may require that you configure and provision an Amazon EC2 instance first\. For example, you might want to test that everything works the way you intend\. There are multiple properties required to create an EC2 instance, such as the AMI ID, instance type, key pair, and security group\. All this information is also required by Amazon EC2 Auto Scaling to launch instances on your behalf when there is a need to scale\. This information is stored in a launch template or launch configuration\. 

You can create an Auto Scaling group using an existing EC2 instance in one of three ways\.
+ One option is to create a launch template from an existing EC2 instance\. Then use the launch template to create a new Auto Scaling group\. For this procedure, see [Creating a launch template from an existing instance \(console\)](create-launch-template.md#create-launch-template-from-instance)\.
+ You can use the console to create an Auto Scaling group from a running EC2 instance\. When you do this, Amazon EC2 Auto Scaling creates a launch configuration for you and associates it with the Auto Scaling group\. This method works well if you want to add the instance to the new Auto Scaling group where it can be managed by Amazon EC2 Auto Scaling\. For more information, see [Attach EC2 instances to your Auto Scaling group](attach-instance-asg.md)\. 
+ The third method, and the subject of the following procedure, is to specify the ID of an existing EC2 instance in the API call that creates the Auto Scaling group\. When you specify an ID of an instance, Amazon EC2 Auto Scaling creates a launch configuration for you and associates it with the Auto Scaling group\. This launch configuration has the same name as the Auto Scaling group, and it derives its attributes from the specified instance, such as AMI ID, instance type, key pair, and security group\. 

**Limitations**  
The following are limitations when using the following procedure to create an Auto Scaling group from an EC2 instance:
+ If the identified instance has tags, the tags are not copied to the `Tags` attribute of the new Auto Scaling group\.
+ The Auto Scaling group includes the block device mapping from the AMI used to launch the instance\. It does not include any block devices attached after instance launch\.
+ If the identified instance is registered with one or more load balancers, the information about the load balancer is not copied to the load balancer or target group attribute of the new Auto Scaling group\.

**Prerequisites**

Before you begin, find the ID of the EC2 instance using the Amazon EC2 console or the [https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html) command \(AWS CLI\)\. The EC2 instance must meet the following criteria:
+ The instance is in the subnet and Availability Zone in which to create the Auto Scaling group\.
+ The instance is not a member of another Auto Scaling group\.
+ The instance is in the `running` state\.
+ The AMI used to launch the instance must still exist\.

**Topics**
+ [Create an Auto Scaling group from an EC2 instance \(AWS CLI\)](#create-asg-from-instance-aws-cli)

## Create an Auto Scaling group from an EC2 instance \(AWS CLI\)<a name="create-asg-from-instance-aws-cli"></a>

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group, *my\-asg\-from\-instance*, from the EC2 instance `i-7f12e649`\.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg-from-instance \
  --instance-id i-7f12e649 --min-size 1 --max-size 2 --desired-capacity 2
```

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to describe the Auto Scaling group\.

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg-from-instance
```

The following example response shows that the desired capacity of the group is 2, the group has 2 running instances, and the launch configuration is named *my\-asg\-from\-instance*\.

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupARN": "arn",
            "HealthCheckGracePeriod": 0,
            "SuspendedProcesses": [],
            "DesiredCapacity": 2,
            "Tags": [],
            "EnabledMetrics": [],
            "LoadBalancerNames": [],
            "AutoScalingGroupName": "my-asg-from-instance",
            "DefaultCooldown": 300,
            "MinSize": 1,
            "Instances": [
                {
                    "InstanceId": "i-6bd79d87",
                    "AvailabilityZone": "us-west-2a",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService",
                    "LaunchConfigurationName": "my-asg-from-instance"
                },
                {
                    "InstanceId": "i-6cd79d80",
                    "AvailabilityZone": "us-west-2a",
                    "HealthStatus": "Healthy",
                    "LifecycleState": "InService",
                    "LaunchConfigurationName": "my-asg-from-instance"
                }
            ],
            "MaxSize": 2,
            "VPCZoneIdentifier": "subnet-6bea5f06",
            "TerminationPolicies": [
                "Default"
            ],
            "LaunchConfigurationName": "my-asg-from-instance",
            "CreatedTime": "2014-12-29T16:14:50.397Z",
            "AvailabilityZones": [
                "us-west-2a"
            ],
            "HealthCheckType": "EC2"
        }
    ]
}
```

Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to describe the launch configuration *my\-asg\-from\-instance*\.

```
aws autoscaling describe-launch-configurations --launch-configuration-names my-asg-from-instance
```