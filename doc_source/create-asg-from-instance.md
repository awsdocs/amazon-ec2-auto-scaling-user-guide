# Creating an Auto Scaling group using an EC2 instance<a name="create-asg-from-instance"></a>

Creating an Auto Scaling group might require that you configure and provision an Amazon EC2 instance first\. For example, you might want to test that everything works the way that you intend\. Multiple properties are required to create an EC2 instance, such as the AMI ID, instance type, key pair, and security group\. All of this information is also required by Amazon EC2 Auto Scaling to launch instances on your behalf when there is a need to scale\. This information is stored in a launch template or launch configuration\. 

You can create an Auto Scaling group using an existing EC2 instance in one of three ways\.
+ Create a launch template from an existing EC2 instance\. Then use the launch template to create a new Auto Scaling group\. For this procedure, see [Creating a launch template from an existing instance \(console\)](create-launch-template.md#create-launch-template-from-instance)\.
+ Use the console to create an Auto Scaling group from a running EC2 instance\. When you do this, Amazon EC2 Auto Scaling creates a launch configuration for you and associates it with the Auto Scaling group\. This method works well if you want to add the instance to the new Auto Scaling group where it can be managed by Amazon EC2 Auto Scaling\. For more information, see [Attach EC2 instances to your Auto Scaling group](attach-instance-asg.md)\. 
+ Specify the ID of an existing EC2 instance in the API call that creates the Auto Scaling group\. This method is the subject of the following procedure\.

When you specify an ID of an existing instance, Amazon EC2 Auto Scaling creates a launch configuration for you and associates it with the Auto Scaling group\. This launch configuration has the same name as the Auto Scaling group, and it derives its attributes from the specified instance, including the AMI ID, instance type, key pair, and security group\. The block devices come from the AMI that was used to launch the instance\. 

## Limitations and prerequisites<a name="create-asg-from-instance-limitations"></a>

The following are limitations when using the following procedure to create an Auto Scaling group from an EC2 instance:
+ If the identified instance has tags, the tags are not copied to the `Tags` attribute of the new Auto Scaling group\.
+ The Auto Scaling group includes the block devices from the AMI that was used to launch the instance\. It does not include any block devices that were attached after instance launch\.
+ If the identified instance is registered with one or more load balancers, the information about the load balancer is not copied to the load balancer or target group attribute of the new Auto Scaling group\.

Before you begin, find the ID of the EC2 instance using the Amazon EC2 console or the [describe\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html) command \(AWS CLI\)\. The EC2 instance must meet the following criteria:
+ The instance is in the subnet and Availability Zone in which you want to create the Auto Scaling group\.
+ The instance is not a member of another Auto Scaling group\.
+ The instance is in the `running` state\.
+ The AMI that was used to launch the instance must still exist\.

## Create an Auto Scaling group from an EC2 instance \(AWS CLI\)<a name="create-asg-from-instance-aws-cli"></a>

The following examples show how to use the AWS CLI to create an Auto Scaling group from an EC2 instance\.

**To create an Auto Scaling group from an EC2 instance**
+ Use the following [create\-auto\-scaling\-group](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create an Auto Scaling group, `my-asg-from-instance`, from the EC2 instance `i-0e69cc3f05f825f4f`\.

  ```
  aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg-from-instance \
    --instance-id i-0e69cc3f05f825f4f --min-size 1 --max-size 2 --desired-capacity 2
  ```

**To verify that your Auto Scaling group has launched instances**
+ Use the following [describe\-auto\-scaling\-groups](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify that the Auto Scaling group was created successfully\.

  ```
  aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg-from-instance
  ```

  The following example response shows that the desired capacity of the group is 2, the group has 2 running instances, and the launch configuration is named `my-asg-from-instance`\.

  ```
  {
    "AutoScalingGroups":[
      {
        "AutoScalingGroupName":"my-asg-from-instance",
        "AutoScalingGroupARN":"arn",
        "LaunchConfigurationName":"my-asg-from-instance",
        "MinSize":1,
        "MaxSize":2,
        "DesiredCapacity":2,
        "DefaultCooldown":300,
        "AvailabilityZones":[
          "us-west-2a"
        ],
        "LoadBalancerNames":[ ],
        "TargetGroupARNs":[ ],
        "HealthCheckType":"EC2",
        "HealthCheckGracePeriod":0,
        "Instances":[
          {
            "InstanceId":"i-06905f55584de02da",
            "InstanceType":"t2.micro",
            "AvailabilityZone":"us-west-2a",
            "LifecycleState":"InService",
            "HealthStatus":"Healthy",
            "LaunchConfigurationName":"my-asg-from-instance",
            "ProtectedFromScaleIn":false
          },
          {
            "InstanceId":"i-087b42219468eacde",
            "InstanceType":"t2.micro",
            "AvailabilityZone":"us-west-2a",
            "LifecycleState":"InService",
            "HealthStatus":"Healthy",
            "LaunchConfigurationName":"my-asg-from-instance",
            "ProtectedFromScaleIn":false
          }
        ],
        "CreatedTime":"2020-10-28T02:39:22.152Z",
        "SuspendedProcesses":[ ],
        "VPCZoneIdentifier":"subnet-6bea5f06",
        "EnabledMetrics":[ ],
        "Tags":[ ],
        "TerminationPolicies":[
          "Default"
        ],
        "NewInstancesProtectedFromScaleIn":false,
        "ServiceLinkedRoleARN":"arn"
      }
    ]
  }
  ```

**To view the launch configuration**
+ Use the following [describe\-launch\-configurations](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-launch-configurations.html) command to view the details of the launch configuration\.

  ```
  aws autoscaling describe-launch-configurations --launch-configuration-names my-asg-from-instance
  ```

  The following is example output:

  ```
  {
    "LaunchConfigurations":[
      {
        "LaunchConfigurationName":"my-asg-from-instance",
        "LaunchConfigurationARN":"arn",
        "ImageId":"ami-0528a5175983e7f28",
        "KeyName":"my-key-pair-uswest2",
        "SecurityGroups":[
          "sg-05eaec502fcdadc2e"
        ],
        "ClassicLinkVPCSecurityGroups":[ ],
        "UserData":"",
        "InstanceType":"t2.micro",
        "KernelId":"",
        "RamdiskId":"",
        "BlockDeviceMappings":[ ],
        "InstanceMonitoring":{
          "Enabled":true
        },
        "CreatedTime":"2020-10-28T02:39:22.321Z",
        "EbsOptimized":false,
        "AssociatePublicIpAddress":true
      }
    ]
  }
  ```

**To terminate the instance**
+ If you no longer need the instance, you can terminate it\. The following [terminate\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html) command terminates the instance `i-0e69cc3f05f825f4f`\. 

  ```
  aws ec2 terminate-instances --instance-ids i-0e69cc3f05f825f4f
  ```

  After you terminate an Amazon EC2 instance, you can't restart the instance\. After termination, its data is gone and the volume can't be attached to any instance\. To learn more about terminating instances, see [Terminating an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html#terminating-instances-console) in the *Amazon EC2 User Guide for Linux Instances*\.