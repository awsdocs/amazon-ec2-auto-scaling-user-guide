# Configure an Application Load Balancer or Network Load Balancer from the Amazon EC2 Auto Scaling console<a name="as-create-load-balancer-console"></a>

Use the following procedure to create and attach an Application Load Balancer or a Network Load Balancer as you create your Auto Scaling group\. 

**To create and attach a new load balancer as you create a new Auto Scaling group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/), and choose **Auto Scaling Groups** from the navigation pane\.

1. Choose **Create Auto Scaling group**\.

1. In steps 1 and 2, choose the options as desired and proceed to **Step 3: Configure advanced options**\.

1. For **Load balancing**, choose **Attach to a new load balancer**\.

   1. Under **Attach to a new load balancer**, for **Load balancer type**, choose whether to create an Application Load Balancer or Network Load Balancer\. 

   1. For **Load balancer name**, enter a name for the load balancer, or keep the default name\.

   1. For **Load balancer scheme**, choose whether to create a public internet\-facing load balancer, or keep the default for an internal load balancer\.

   1. For **Availability Zones and subnets**, select the public subnet for each Availability Zone in which you chose to launch your EC2 instances\. \(These prepopulate from step 2\.\)\.

   1. For **Listeners and routing**, update the port number for your listener \(if necessary\), and under **Default routing**, choose **Create a target group**\. Alternatively, you can choose an existing target group from the drop\-down list\.

   1. If you chose **Create a target group** in the last step, for **New target group name**, enter a name for the target group, or keep the default name\. 

   1. To add tags to your load balancer, choose **Add tag**, and provide a tag key and value for each tag\.

1. Proceed to create the Auto Scaling group\. Your instances will be automatically registered to the load balancer after the Auto Scaling group has been created\. 
**Note**  
After creating your Auto Scaling group, you can use the Elastic Load Balancing console to create additional listeners\. This is useful if you need to create a listener with a secure protocol, such as HTTPS, or a UDP listener\. You can add more listeners to existing load balancers, as long as you use distinct ports\.