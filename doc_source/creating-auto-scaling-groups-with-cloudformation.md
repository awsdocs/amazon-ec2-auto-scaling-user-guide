# Create Auto Scaling groups with AWS CloudFormation<a name="creating-auto-scaling-groups-with-cloudformation"></a>

Amazon EC2 Auto Scaling is integrated with AWS CloudFormation, a service that helps you to model and set up your AWS resources so that you can spend less time creating and managing your resources and infrastructure\. You create a template that describes all the AWS resources that you want \(such as Auto Scaling groups\), and AWS CloudFormation provisions and configures those resources for you\. 

When you use AWS CloudFormation, you can reuse your template to set up your Amazon EC2 Auto Scaling resources consistently and repeatedly\. Describe your resources once, and then provision the same resources over and over in multiple AWS accounts and Regions\. 

## Amazon EC2 Auto Scaling and AWS CloudFormation templates<a name="working-with-templates"></a>

To provision and configure resources for Amazon EC2 Auto Scaling and related services, you must understand [AWS CloudFormation templates](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html)\. Templates are formatted text files in JSON or YAML\. These templates describe the resources that you want to provision in your AWS CloudFormation stacks\. If you're unfamiliar with JSON or YAML, you can use AWS CloudFormation Designer to help you get started with AWS CloudFormation templates\. For more information, see [What is AWS CloudFormation Designer?](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/working-with-templates-cfn-designer.html) in the *AWS CloudFormation User Guide*\.

To get started creating your own stack templates for Amazon EC2 Auto Scaling, complete the following tasks:
+ Create a launch template using [AWS::EC2::LaunchTemplate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-launchtemplate.html)\. 
+ Create an Auto Scaling group using [AWS::AutoScaling::AutoScalingGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-group.html)\. 

For a walkthrough that shows you how to deploy an Auto Scaling group behind an Application Load Balancer, see [Walkthrough: Create a scaled and load\-balanced application](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/example-templates-autoscaling.html) in the *AWS CloudFormation User Guide*\.

You can find additional useful examples of templates that create Auto Scaling groups and related resources in the [Auto scaling template snippets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-autoscaling.html) section of the *AWS CloudFormation User Guide*\. For more information and example snippets, see the [Amazon EC2 Auto Scaling resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_AutoScaling.html) in the *AWS CloudFormation User Guide*\.

## Learn more about AWS CloudFormation<a name="learn-more-cloudformation"></a>

To learn more about AWS CloudFormation, see the following resources:
+ [AWS CloudFormation](http://aws.amazon.com/cloudformation/)
+ [AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
+ [AWS CloudFormation API Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/Welcome.html)
+ [AWS CloudFormation Command Line Interface User Guide](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/what-is-cloudformation-cli.html)