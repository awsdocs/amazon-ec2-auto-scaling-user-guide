# Instance weighting for Amazon EC2 Auto Scaling<a name="asg-instance-weighting"></a>

When you configure an Auto Scaling group to launch multiple instance types, you have the option of defining the number of capacity units that each instance contributes to the desired capacity of the group, using *instance weighting*\. This allows you to specify the relative weight of each instance type in a way that directly maps to the performance of your application\. You can weight your instances to suit your specific application needs, for example, by the cores \(vCPUs\) or by memory \(GiBs\)\. 

For example, let's say that you run a compute\-intensive application that performs best with at least 8 vCPUs and 15 GiB of RAM\. If you use `c5.2xlarge` as your base unit, any of the following EC2 instance types would meet your application needs\. 


**Instance types example**  

| Instance type | vCPU | Memory \(GiB\) | 
| --- | --- | --- | 
| c5\.2xlarge  |  8  | 16 | 
| c5\.4xlarge | 16 | 32 | 
| c5\.12xlarge | 48 | 96 | 
| c5\.18xlarge  | 72 | 144 | 
| c5\.24xlarge | 96 | 192 | 

By default, all instance types are treated as the same weight\. In other words, whether Amazon EC2 Auto Scaling launches a large or small instance type, each instance counts toward the group's desired capacity\. 

With instance weighting, however, you assign a number value that specifies how many capacity units to associate with each instance type\. For example, if the instances are of different sizes, a `c5.2xlarge` instance could have the weight of 2, and a `c5.4xlarge` \(which is two times bigger\) could have the weight of 4, and so on\. Then, when Amazon EC2 Auto Scaling launches instances, their weights count toward your desired capacity\. 

## Price per unit hour<a name="weights-price-per-unit-hour"></a>

The following table compares the hourly price for Spot Instances in different Availability Zones in US East \(N\. Virginia, Ohio\) with the price for On\-Demand Instances in the same Region\. The prices shown are example pricing and not current pricing\. These are your costs *per instance hour*\. 


**Example: Spot pricing per instance hour**  

| Instance type | us\-east\-1a | us\-east\-1b | us\-east\-1c | On\-Demand pricing | 
| --- | --- | --- | --- | --- | 
| c5\.2xlarge  | $0\.180 | $0\.191 | $0\.170 | $0\.34  | 
| c5\.4xlarge | $0\.341 | $0\.361 | $0\.318 | $0\.68 | 
| c5\.12xlarge  | $0\.779 | $0\.777  | $0\.777  | $2\.04 | 
| c5\.18xlarge  | $1\.207 | $1\.475 | $1\.357 | $3\.06 | 
| c5\.24xlarge | $1\.555 | $1\.555 | $1\.555 | $4\.08 | 

With instance weighting, you can evaluate your costs based on what you use *per unit hour*\. You can determine the price per unit hour by dividing your price for an instance type by the number of units that it represents\. For On\-Demand Instances, the price *per unit hour* is the same when deploying one instance type as it is when deploying a different size of the same instance type\. In contrast, however, the Spot price *per unit hour* varies by Spot pool\. 

The easiest way to understand how the price *per unit hour* calculation works with weighted instances is with an example\. For example, for ease of calculation, let's say you want to launch Spot Instances only in us\-east\-1a\. The *per unit hour price* is captured below\.


**Example: Spot Price per unit hour example**  

| Instance type | us\-east\-1a | Instance weight | Price per unit hour  | 
| --- | --- | --- | --- | 
| c5\.2xlarge  | $0\.180 | 2 | $0\.090 | 
| c5\.4xlarge | $0\.341 | 4 | $0\.085 | 
| c5\.12xlarge  | $0\.779 | 12 | $0\.065 | 
| c5\.18xlarge  | $1\.207 | 18 | $0\.067 | 
| c5\.24xlarge | $1\.555 | 24 | $0\.065 | 

## Considerations<a name="weights-considerations"></a>

This section discusses the key considerations in implementing instance weighting effectively\.
+ Start by choosing a few instance types that reflect the actual performance requirements of your application\. Then, decide how much each instance type should count toward the desired capacity of your Auto Scaling group by specifying their weights\. The weights apply to current and future instances in the group\. 
+ Be cautious about choosing very large ranges for your weights\. For example, we don't recommend specifying a weight of 1 for an instance type when the next larger instance type has a weight of 200\. The difference between the smallest and largest weights should also not be extreme\. If any of the instance types have too large of a weight difference, this can have a negative effect on ongoing cost\-performance optimization\. 
+ The size of the Auto Scaling group is measured in capacity units, and not in instances\. For example, if your weights are based on vCPUs, you must specify the desired, minimum, and maximum number of cores you want\.
+ Set your weights and desired capacity so that the desired capacity is at least two to three times larger than your largest weight\. 
+ If you choose to set your own maximum price for Spot, you must specify a price *per instance hour* that is high enough for your most expensive instance type\. Amazon EC2 Auto Scaling provisions Spot Instances if the current Spot price in an Availability Zone is below your maximum price and capacity is available\. If the request for Spot Instances cannot be fulfilled in one Spot Instance pool, it keeps trying in other Spot pools to leverage the cost savings of Spot Instances\. 

With instance weighting, the following new behaviors are introduced:
+ Current capacity will either be at the desired capacity or above it\. Because Amazon EC2 Auto Scaling wants to provision instances until the desired capacity is totally fulfilled, an overage can happen\. For example, suppose that you specify two instance types, `c5.2xlarge` and `c5.12xlarge`, and you assign instance weights of 2 for `c5.2xlarge` and 12 for `c5.12xlarge`\. If there are 5 units remaining to fulfill the desired capacity, and Amazon EC2 Auto Scaling provisions a `c5.12xlarge`, the desired capacity is exceeded by 7 units\. 
+ When Amazon EC2 Auto Scaling provisions instances to reach the desired capacity, distributing instances across Availability Zones and respecting the allocation strategies for On\-Demand and Spot Instances both take precedence over avoiding overages\. 
+ Amazon EC2 Auto Scaling can overstep the maximum capacity limit to maintain balance across Availability Zones, using your preferred allocation strategies\. The hard limit enforced by Amazon EC2 Auto Scaling is a value that is equal to your desired capacity plus your largest weight\.

Note the following when adding or modifying weights for existing groups:
+ When adding instance weights to an existing Auto Scaling group, you must include any instance types that are already running in the group\. 
+ When modifying existing instance weights, Amazon EC2 Auto Scaling will launch or terminate instances to reach your desired capacity based on the new weights\. 
+ If you remove an instance type, any running instances of that instance type will continue to have their last updated weight values, even though the instance type has been removed\.

## Add or modify weights for your Auto Scaling group<a name="add-weights"></a>

You can add weights to an existing Auto Scaling group, or to a new Auto Scaling group as you create it\. You can also update an existing Auto Scaling group to define new configuration options \(Spot/On\-Demand usage, Spot allocation strategy, instance types\)\. If you change how many Spot or On\-Demand Instances you want, Amazon EC2 Auto Scaling gradually replaces existing instances to match the new purchase options\. 

Before creating Auto Scaling groups using instance weighting, we recommend that you become familiar with launching groups with multiple instance types\. For more information and additional examples, see [Auto Scaling groups with multiple instance types and purchase options](asg-purchase-options.md)\.

The following examples show how to use the AWS CLI to add weights when you create Auto Scaling groups, and to add or modify weights for existing Auto Scaling groups\. You can configure a variety of parameters in a JSON file, and then reference the JSON file as the sole parameter for your Auto Scaling group\. 

**To add weights to an Auto Scaling group on creation**
+ Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/create-auto-scaling-group.html) command to create a new Auto Scaling group\. For example, the following command creates a new Auto Scaling group and adds instance weighting by specifying the following:
  + The percentage of the group to launch as On\-Demand Instances \(`0`\) and a base number of On\-Demand Instances to start with \(`10`\)
  + The allocation strategy for Spot Instances in each Availability Zone \(`capacity-optimized`\)
  + The instance types to launch in priority order \(`m4.16xlarge`, `m5.24xlarge`\)
  + The instance weights that correspond to the relative size difference \(vCPUs\) between instance types \(`16`, `24`\)
  + The subnets in which to launch the instances \(`subnet-5ea0c127`, `subnet-6194ea3b`, `subnet-c934b782`\), each corresponding to a different Availability Zone
  + The launch template \(`my-launch-template`\) and the launch template version \(`$Latest`\)

  ```
  aws autoscaling create-auto-scaling-group --cli-input-json file://~/config.json
  ```

  The following is an example `config.json` file\. 

  ```
  {
      "AutoScalingGroupName": "my-asg",
      "MixedInstancesPolicy": {
          "LaunchTemplate": {
              "LaunchTemplateSpecification": {
                  "LaunchTemplateName": "my-launch-template",
                  "Version": "$Latest"
              },
              "Overrides": [
                  {
                      "InstanceType": "m4.16xlarge",
                      "WeightedCapacity": "16"
                  },
                  {
                      "InstanceType": "m5.24xlarge",
                      "WeightedCapacity": "24"
                  }
              ]
          },
          "InstancesDistribution": {
              "OnDemandBaseCapacity": 10,
              "OnDemandPercentageAboveBaseCapacity": 0,
              "SpotAllocationStrategy": "capacity-optimized"
          }
      },
      "MinSize": 160,
      "MaxSize": 720,
      "DesiredCapacity": 480,
      "VPCZoneIdentifier": "subnet-5ea0c127,subnet-6194ea3b,subnet-c934b782",
      "Tags": []
  }
  ```

**To add or modify weights for an existing Auto Scaling group**
+ Use the [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command to add or modify weights\. For example, the following command adds weights to instance types in an existing Auto Scaling group by specifying the following:
  + The instance types to launch in priority order \(`c5.18xlarge`, `c5.24xlarge`, `c5.2xlarge`, `c5.4xlarge`\)
  + The instance weights that correspond to the relative size difference \(vCPUs\) between instance types \(`18`, `24`, `2`, `4`\)
  + The new, increased desired capacity, which is larger than the largest weight

  ```
  aws autoscaling update-auto-scaling-group --cli-input-json file://~/config.json
  ```

  The following is an example `config.json` file\. 

  ```
  {
      "AutoScalingGroupName": "my-existing-asg",
      "MixedInstancesPolicy": {
          "LaunchTemplate": {
              "Overrides": [
                  {
                      "InstanceType": "c5.18xlarge",
                      "WeightedCapacity": "18"
                  },
                  {
                      "InstanceType": "c5.24xlarge",
                      "WeightedCapacity": "24"
                  },
                  {
                      "InstanceType": "c5.2xlarge",
                      "WeightedCapacity": "2"
                  },
                  {
                      "InstanceType": "c5.4xlarge",
                      "WeightedCapacity": "4"
                  }
              ]
          }
      },
      "MinSize": 0,
      "MaxSize": 100,
      "DesiredCapacity": 100
  }
  ```

**To verify the weights for an Auto Scaling group**
+ Use the following [https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command to verify the weights\.

  ```
  aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
  ```

  The following is an example response\.

  ```
  {
      "AutoScalingGroups": [
          {
              "AutoScalingGroupName": "my-asg",
              "AutoScalingGroupARN": "arn",
              "MixedInstancesPolicy": {
                  "LaunchTemplate": {
                      "LaunchTemplateSpecification": {
                          "LaunchTemplateId": "lt-0b97f1e282EXAMPLE",
                          "LaunchTemplateName": "my-launch-template",
                          "Version": "$Latest"
                      },
                      "Overrides": [
                          {
                              "InstanceType": "m4.16xlarge",
                              "WeightedCapacity": "16"
                          },
                          {
                              "InstanceType": "m5.24xlarge",
                              "WeightedCapacity": "24"
                          }
                      ]
                  },
                  "InstancesDistribution": {
                      "OnDemandAllocationStrategy": "prioritized",
                      "OnDemandBaseCapacity": 10,
                      "OnDemandPercentageAboveBaseCapacity": 0,
                      "SpotAllocationStrategy": "capacity-optimized"
                  }
              },
              "MinSize": 160,
              "MaxSize": 720,
              "DesiredCapacity": 480,
              "DefaultCooldown": 300,
              "AvailabilityZones": [
                  "us-west-2a",
                  "us-west-2b",
                  "us-west-2c"
              ],
              "LoadBalancerNames": [],
              "TargetGroupARNs": [],
              "HealthCheckType": "EC2",
              "HealthCheckGracePeriod": 0,
              "Instances": [
                  {
                      "InstanceId": "i-027327f0ace86f499",
                      "InstanceType": "m5.24xlarge",
                      "AvailabilityZone": "us-west-2a",
                      "LifecycleState": "InService",
                      "HealthStatus": "Healthy",
                      "LaunchTemplate": {
                          "LaunchTemplateId": "lt-0b97f1e282EXAMPLE",
                          "LaunchTemplateName": "my-launch-template",
                          "Version": "7"
                      },
                      "ProtectedFromScaleIn": false,
                      "WeightedCapacity": "24"
                  },
                  {
                      "InstanceId": "i-0ec0d761cc134878d",
                      "InstanceType": "m4.16xlarge",
                      "AvailabilityZone": "us-west-2a",
                      "LifecycleState": "Pending",
                      "HealthStatus": "Healthy",
                      "LaunchTemplate": {
                          "LaunchTemplateId": "lt-0b97f1e282EXAMPLE",
                          "LaunchTemplateName": "my-launch-template",
                          "Version": "7"
                      },
                      "ProtectedFromScaleIn": false,
                      "WeightedCapacity": "16"
                  },
             ...
          }
      ]
  }
  ```