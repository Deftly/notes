# Virtual Private Cloud(VPC)
A VPC is a logically isolated portion of the AWS cloud in which you can deploy your own AWS resources in a private space. This affords control over the networking environment, including the address ranges used, routing configuration, security groups for instances and firewalls for subnets. There are also various ways that you can connect to and from your VPCs and technologies to interconnect your various VPCs. 

## AWS Global Infrastructure
![aws_global_infrastructure](./assets/aws_global_infrastructure.png)

A **Region** is a physical location in the world that is independent and geographically distant from other regions. Within regions with have **Availability Zones**(AZ) which are composed of one or more data centers. AWS regions are connected via a high bandwidth, fully redundant network. Each region consists for two or more AZs, though some have as many as 6. **Local Zones** extend regions closer to end-users, this is mainly used to reduce latency. 

There may be situations where we want to extend some AWS services into an on-premises data center, this can be done using **AWS Outposts**. Outposts allow the operation of some AWS services on dedicated hardware within your own data center, this provides very low latency access to those services. 

**AWS Wavelength Zone** allows users to connect, with low latency, over 5G to services being run in the wavelength zone. Wavelength zones, outposts, and local zones are all connected to the AWS region.

### CloudFront Network
![cloudfront_network](./assets/cloudfront_network.png)

**Regional Edge Caches** and **Edge locations** are part of the CloudFront Network, which is Contend Delivery Network(CDN) service. This is used to get content closer to end users for better performance. 

## IPv4 Addressing Primer
![dns_resolution](./assets/dns_resolution.png)

### Structure of an IPv4 Address
IP addresses are written in dotted decimal notation `192.168.0.1`, each part of the address is a binary octet.

### Networks and Hosts
Every IP address has a network ID and a host ID. The network ID will be the same for every computer on a particular network, the host ID will be unique for each computer on the network. The way we know which portion is the network ID and which is the host ID is through a subnet mask. Every bit in the subnet mask that is 1 is a value that are used for the network id and every bit that is a 0 is used for a host id.

![network_and_host_id](./assets/network_and_host_id.png)

![subnet_mask_ex](./assets/subnet_mask_ex.png)

