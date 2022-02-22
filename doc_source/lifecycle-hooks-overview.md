# How lifecycle hooks work<a name="lifecycle-hooks-overview"></a>

An Amazon EC2 instance transitions through different states from the time it launches until it is terminated\. You can create lifecycle hooks to act when an instance transitions into a wait state\.

The following illustration shows the transitions between Auto Scaling instance states\. 

![\[The lifecycle of instances using lifecycle hooks.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/lifecycle_hooks.png)

As shown in the preceding diagram:

1. The Auto Scaling group responds to a scale\-out event and begins launching an instance\.

1. The lifecycle hook puts the instance into a wait state \(`Pending:Wait`\) and then performs a custom action\.

   The instance remains in a wait state until you either complete the lifecycle action using the [complete\-lifecycle\-action](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/complete-lifecycle-action.html) CLI command or the [CompleteLifecycleAction](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CompleteLifecycleAction.html) action, or the timeout period ends\. By default, the instance remains in a wait state for one hour, and then the Auto Scaling group continues the launch process \(`Pending:Proceed`\)\. If you need more time, you can restart the timeout period by recording a heartbeat\. If you complete the lifecycle action before the timeout period expires, the period ends and the Auto Scaling group continues the launch process\.

1. The instance enters the `InService` state and the health check grace period starts\. If the Auto Scaling group is associated with an Elastic Load Balancing load balancer, the instance is registered with the load balancer, and the load balancer starts checking its health\. After the health check grace period ends, Amazon EC2 Auto Scaling begins checking the health state of the instance\.

1. The Auto Scaling group responds to a scale\-in event and begins terminating an instance\. If the Auto Scaling group is being used with Elastic Load Balancing, the terminating instance is first deregistered from the load balancer\. If connection draining is enabled for the load balancer, the instance stops accepting new connections and waits for existing connections to drain before completing the deregistration process\.

1. The lifecycle hook puts the instance into a wait state \(`Terminating:Wait`\) and then performs a custom action\.

   The instance remains in a wait state either until you complete the lifecycle action, or until the timeout period ends \(one hour by default\)\. After you complete the lifecycle hook or the timeout period expires, the instance transitions to the next state \(`Terminating:Proceed`\)\.

1. The instance is terminated\.