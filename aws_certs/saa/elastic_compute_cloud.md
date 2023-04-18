# Amazon Elastic Compute Cloud(EC2)
One of the original and most important services on AWS. It enables the running of virtual servers in the cloud. It's also used by many other services in AWS so it's very important to have a solid understanding of it before moving on.

Within an AWS data center there are host servers on which we can run EC2 instances using server virtualization. The EC2 hosts are managed by AWS and users manage the EC2 instance that is run on top of the host servers.

When launching instances we have to choose an instance type(hardware profile), which designates different combinations of CPU, memory, storage, and networking. This determines both the cost and the performance capabilities of that instance. Afterwards we select an Amazon Machine Image(AMI) which is essentially an image that contains the operating system and any configuration settings. AMIs are created from EBS Snapshots, which are point in time backups of an instance. 

### Benefits of EC2
- Elastic computing: easily launch hundreds or thousands of instances within minutes
- Complete control: administrative access to all instances
- Flexible: choice of instance types, operating systems, and software packages
- Reliable: Offers very high levels of availability and instances can be rapidly commissioned and replaced
- Secure: Fully integrated with Amazon VPC and other security features
- Inexpensive: (Relatively)Low cost, pay for what you use

## Amazon EC2 Instance in a Public Subnet
![ec2_public_subnet](./assets/ec2_public_subnet.png)

Within a region we have a virtual private cloud(VPC) within which we have our resources. Resources within the VPC are private but accessibility can be public. VPCs contain multiple Availability Zones(AZ) and within an AZ we have public and private subnets. Public subnets can receive connections from the internet.

Each instance has a hard drive, in this case a virtual hard drive, known as an Elastic Block Store(EBS) Volume where the data is stored. We also have a security group which determines which ports, protocols, and IP addresses we can connect from. This functions as a firewall securing access to our instances. Security groups can be used to control both inbound and outbound traffic.

We also need an Internet Gateway, this is attached to a VPC and enables access to and from the internet. When we want to connect to our EC2 instance we actually connect to the Internet Gateway and from there to the EC2 instance. 

## EC2 User Data
User data is code that runs when the instance starts up for the first time. The following is an example that installs a web server on our instance:

```Shell
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
```

User data is limited to 16KB and can be uploaded either as text or as a file.

## EC2 Metadata
This is information available about a particular EC2 instance and it is available at the following address: http://169.254.169.254/latest/meta-data. This is a local address on your EC2 instance which you can use to find a variety of information about the instance itself. Here's an example:

```Shell
[ec2-user@ip-172-31-42-248 ~]$ curl http://169.254.169.254/latest/meta-data
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hibernation/
hostname
indentity-credentials/
instance-action
instance-id
instance-life-cycle
instance-type
local-hostname
local-ipv4
```

## Accessing Services - Access Keys and IAM Roles
There will often be occasions where EC2 instances need to connect to other services. For instance, saving data to a storage service or database. There are few ways to do this, the first being access keys. 

Access keys are stored on the instance itself and are associated with a user account that has a permissions policy. This gives the instance access to whatever operations are allowed by that permissions policy. The downside to this approach is that access keys are stored on the file system of the instance which isn't very secure.

A better way to do this is to use an IAM role. We can use an instance profile to connect an IAM role to an EC2 instance. The instance will gain access to whatever permissions are associated with the role provides. The great thing about this is that no credentials are stored on the EC2 instance. 

## EC2 Placement Groups
EC2 placement groups are a way to control how and where AWS deploys EC2 instances within and across AZs. 

### Cluster
This packs instances closely together inside an AZ. This enables workloads to achieve low latency network performance and high throughput for inter-node communication which is often required for High Performance Computing(HPC) applications.

### Partition
This spreads your instances across logical partitions such that groups of instances in different partitions don't share underlying hardware. This is typically used for large distributed and replicated workloads such as Hadoop, Cassandra, and Kafka.

### Spread
This strictly places a small group of instances across distinct underlying hardware to reduce correlated failures.

## Network Interfaces(ENI, ENA, EFA)
![ec2_network_interfaces](./assets/ec2_network_interfaces.png)

When EC2 instances are launched a subnet is chosen, an interface is then attached to that subnet. The actual instance itself isn't in the subnet it is within the AZ, this means it can be attached to multiple subnets through different adapters. In the above example the instance has an eth0 adapter in a public subnet with both a public and private IP address, and an eth1 adapter in a private subnet within the same AZ. It is not possible to attach an adapter from a subnet in a different AZ. 

### Elastic network interface(ENI)
- ENI is the basic adapter for when there are no specific high-performance requirements 
- Can be used with all instance types

### Elastic Network Adapter(ENA)
- ENA provides enhanced networking performance and provides much higher bandwidth and lower inter-instance latency
- It can only be used with supported instance types

### Elastic Fabric Adapter(EFA)
- EFA are used for HPC and ML use cases
- Tightly coupled applications
- Can be used with all instance types

## Public, Private and Elastic IP(EIP) Addresses
Public IP addresses are dynamic addresses, meaning they might change. When you stop your instance and start it back up again it will pick up a new IPV4 public address, the private address will stay the same. This means you should not use the public IP address in application code because it can change over time. 

An Elastic IP is also a public IP but it's a static address and doesn't change and we can associate an Elastic IP with a network interface.

In the event that an instance fails and we have applications that are pointing to it's IP address we will want to is assign that IP address to another instance. To do this we have a two options. We can move the network interface and assign it to another instance within the same AZ. Another option is to move the Elastic IP address to a new instance with its own network interface. This allows you to fail over into a different AZ since the Elastic IP address can be remapped across AZs but the network interfaces cannot. 

Also note that EIP addresses that are allocated to your account but unused will incur charges.
