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

