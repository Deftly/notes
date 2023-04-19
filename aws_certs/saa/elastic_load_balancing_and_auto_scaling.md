# Elastic Load Balancing, and Auto Scaling
Now that we are familiar with EC2 we need to understand how can we make sure that our applications have enough EC2 instances available and distribute incoming connections to those EC2 instances. Two technologies that really help with this are auto scaling and elastic load balancing, working together these two technologies enable elastic and fault tolerant applications.

## EC2 Auto Scaling
![ec2_auto_scaling_group](./assets/ec2_auto_scaling_group.png)

In the diagram above we have an auto scaling group(ASG). Within that group we have a few EC2 instances in public subnets and multiple AZs. When multiple AZs are specified for the ASG the service will generally try and spread the instances evenly across the AZs. 

There are different types of monitoring with EC2 instances. There are metrics being sent to CloudWatch and metrics from the instances themselves. Should an instance fail, the ASG will automatically replace that instance. 

In another scenario if the aggregate CPU utilization across our instances exceeds a certain percentage we can have a can have CloudWatch alarm notify the ASG to launch additional instances. With auto scaling we can respond dynamically to changes in events, scaling out and in as needed. Scaling policies define how to respond to changes in demand. 

### Configuration of an Auto Scaling Group
A Launch Template specifies the configuration of the instances that will be launched within the ASG, this includes things like the AMI, instance type, the volumes to be attached, what security groups to use, what key pair, etc.

Another option is to use a Launch Configuration but these have fewer attributes than, and are being replaced launch templates.

Once we have a launch template or launch configuration we start creating the ASG:
- Configure the purchase options: On-Demand vs Spot, etc.
- Configure the VPC and Subnets; not ASG do not span VPCs always within a single VPC
- Attach a load balancer
- Configure health checks(EC2 and ELB), EC2 checks enabled by default
- Group size and scaling policies

### Health Check Grace Period
- How long to wait before checking the health status of the instance
- Auto Scaling does not act on health checks until grace period expires

### Auto Scaling - Monitoring
#### Group Metrics(ASG)
- Data points about the ASG
- 1-minute granularity
- No charge
- Must be enabled

#### Basic monitoring(Instances)
- 5-minute granularity
- No charge

#### Detailed monitoring(Instances)
- 1-minute granularity
- Charges apply

### Additional Scaling Settings
**Cooldowns** - Used with simple scaling policy to prevent auto scaling from launching or terminating before effects of previous activities are visible. Default value is 5 minutes.

**Termination Policy** - Controls which instances to terminate first when a scale-in event occurs.

**Termination Protection** - Prevents Auto Scaling from terminating protected instances.

**Standby State** - Used to put an instance in the *InService* state into the *Standby* state, this gives an opportunity to update or troubleshoot the instance.

**Lifecycle Hooks** - Used to perform custom actions by pausing instances as the ASG launches or terminates them. These have a few potential use cases:
  - Run a script to download and install software after launching but you are unsure on the time needed so the health check grace period may not be enough. With lifecycle hooks you can pause the instance until they receive a confirmation from the script that the processes have completed.
  - Pause an instance to process data before a scale-in(termination)

## Types of Elastic Load Balancers
There are few different types of Elastic Load Balancers on AWS and it's important to know in which cases to use each one.

### Application Load Balancer(ALB)
- Operates at the request level(layer 7)
- Supports path-based routing, host-based routing, query string parameter-based routing, and source IP address-based routing
- Supports instances, IP addresses, Lambda functions and containers as targets

### Network Load Balancer(NLB)
- Operates at the connection level(layer 4)
- Routes connections based on IP protocol data
- Offers ultra high performance, low latency and TLS offloading at scale
- Can have a static IP/Elastic IP
- Supports UDP and static IP addresses as targets 

### Gateway Load Balancer(GLB)
- Used in front of virtual appliances such as firewalls, Intrusion Detection Systems(IDS)/Intrusion Prevention Systems(IPS), and deep packet inspection systems
- Operates at Layer 3 - listens for packets on all ports
- Forwards traffic to the Target Group(TG) specified in the listener rules
- Exchanges traffic with appliances using the GENEVE protocol on port 6081

## Routing with ALB and NLB

### ALB
![alb_routing](./assets/alb_routing.png)

### NLB
![nlb_routing](./assets/nlb_routing.png)

## EC2 Scaling Policies

### Dynamic Scaling - Target Tracking
![target_tracking](./assets/target_tracking.png)

In the above example we have an ASG with 4 instances and we are tracking the ASGAverageCPUUtilization. Once CloudWatch has reported that the Average CPU exceeds our limit it will try to get it back under the threshold. Instance metrics are not counted until the *warm up* time has expired. When using target tracking AWS recommends scaling up metrics with a 1 minute frequency.

### Dynamic Scaling - Simple Scaling
![simple_scaling](./assets/simple_scaling.png)

With Simple scaling the ASG has an alarm set to some metric, in this case CPU utilization greater than or equal to 60%. When the alarm triggers it will launch 2 new instances then wait 300 seconds before allowing another scaling activity.

### Dynamic Scaling - Step Scaling
![step_scaling](./assets/step_scaling.png)

Very similar to simple scaling but how far we exceed the metric for which we have an alarm will trigger different responses. In this example if we exceed 60% CPU utilization by 20% we will launch four new instances instead of just two. 

### Scheduled Scaling
![scheduled_scaling](./assets/scheduled_scaling.png)

With scheduled scaling we can set a time at which the ASG will scale and how many instances we want it to scale by. When configuring a scheduled scaling policy we define a desired, minimum, and maximum number of instances along with the start time and how often it occurs. 

## Cross-Zone Load Balancing
When cross-zone load balancing is enabled each load balancer node distributes traffic across the registered targets in all enabled AZs. When it is disabled each load balancer distributes traffic only across the registered targets in its AZ. ALBs have cross-zone load balancing always enabled while NLBs and GLBs have it disabled by default. Cross zone load balancing enables you to better distribute traffic.

## Session State and Session Stickiness
When using ELBs the applications on the EC2 instances are stateless. Any persistent data that we need to store must be stored outside the EC2 instance. And if a user is redirected to a different instance they shouldn't have to re-authenticate. 

One technique to achieve this is to store session state data externally. Session data such as authentication details can be stored in a DynamoDB table or some other form of external storage. Then if a user is directed to a different instance that authentication information can be retrieved from the external storage so they don't have to log in again.

Another technique is called sticky sessions. When a user connects to an instance a cookie is generated which binds a client to that instance for the lifetime of the cookie. If the client connection gets dropped and they reconnect the cookie will tell the load balancer which instance to connect back to. In this case session state data like authentication details might be stored locally on the instance itself. If the instance fails then the user will be directed somewhere else and the session data will be lost. Sticky sessions can be used with an external store for more resiliency though there is some added latency with that approach.

## Secure Listeners for ELB
Most websites and web applications are secured with SSL/TLS certificates. This means the connections from users through to the web applications are encrypted in transit. With an ELB we can achieve this using a secure listener. The way we enable this is slightly different between ALBs and NLBs.

### ALB Secure listener
![alb_secure_listener](./assets/alb_secure_listener.png)

### NLB Secure Listener
![nlb_secure_listener](./assets/nlb_secure_listener.png)
