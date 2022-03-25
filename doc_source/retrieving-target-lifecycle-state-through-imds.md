# Retrieve the target lifecycle state through instance metadata<a name="retrieving-target-lifecycle-state-through-imds"></a>

**Important**  
Amazon EC2 Auto Scaling started generating the target lifecycle state on March 10, 2022\. If your instance transitions to one of the target lifecycle states after that date, the target lifecycle state item is present in your instance metadata\. Otherwise, it is not present, and you receive an HTTP 404 error\.

Each Auto Scaling instance that you launch goes through several lifecycle states\. If you want to fine tune your on\-instance custom actions to act on specific lifecycle state transitions, you can do this by retrieving the target lifecycle state through instance metadata\. For example, you might create lifecycle hooks for instances in a warm pool so that hibernated or stopped instances can be attached to a cluster on restart\.

The Auto Scaling instance lifecycle has two primary steady states—`InService` and `Terminated`—and two side steady states—`Detached` and `Standby`\. If you use a warm pool, the lifecycle has four additional steady states—`Warmed:Hibernated`, `Warmed:Running`, `Warmed:Stopped`, and `Warmed:Terminated`\.

When an instance prepares to transition to one of the preceding steady states, Amazon EC2 Auto Scaling updates the value of the instance metadata item `autoscaling/target-lifecycle-state`\. To get the target lifecycle state from within the instance, you must use the instance metadata service to retrieve it from the instance metadata\. 

**Note**  
*Instance metadata* is data about an Amazon EC2 instance that applications can use to query instance information\. The *instance metadata service* is an on\-instance component that local code uses to access instance metadata\. Local code can include user data scripts or applications running on the instance\.

Local code can access instance metadata from a running instance using one of two methods: Instance Metadata Service Version 1 \(IMDSv1\) or Instance Metadata Service Version 2 \(IMDSv2\)\. IMDSv2 uses session\-oriented requests and mitigates several types of vulnerabilities that could be used to try to access the instance metadata\. For details about these two methods, see [Use IMDSv2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) in the *Amazon EC2 User Guide for Linux Instances*\.

------
#### [ IMDSv2 ]

```
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` \
&& curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/autoscaling/target-lifecycle-state
```

------
#### [ IMDSv1 ]

```
curl http://169.254.169.254/latest/meta-data/autoscaling/target-lifecycle-state
```

------

The following is example output\.

```
InService
```

The target lifecycle state is the state that the instance is transitioning to\. The current lifecycle state is the state that the instance is in\. These can be the same after the lifecycle action is complete and the instance finishes its transition to the target lifecycle state\. You cannot retrieve the current lifecycle state of the instance from the instance metadata\.

For a tutorial that shows you how to create a lifecycle hook with a custom action in a user data script that uses the target lifecycle state, see [Tutorial: Configure user data to retrieve the target lifecycle state through instance metadata](tutorial-lifecycle-hook-instance-metadata.md)\.

For more information about instance metadata categories that you can use to configure or manage instances, see [Instance metadata categories](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-categories.html) in the *Amazon EC2 User Guide for Linux Instances*\. For more information about retrieving instance metadata, see [Retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) in the *Amazon EC2 User Guide for Linux Instances*\.