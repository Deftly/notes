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

## VPC Overview
A VPC is a logically isolated portion of the AWS cloud within a region that you can deploy resource into. Subnets are created within AZs and you can have multiple subnets within the same AZ but they cannot span multiple AZs. The **VPC router** takes care of all routing for connections that go outside of a subnet and is configured using route tables. In order to connect to the internet we also need an internet gateway, this is attached to a VPC and you only ever have one per VPC. The internet gateway handles both ingress and egress traffic. 

You can create multiple VPCs per region, limited to 5 by default but you can request an increase if necessary. Each VPC has a CIDR block, the overall block of addresses from which you then create the addresses assigned to your subnets.

![multiple_vpcs](./assets/multiple_vpcs.png)

### VPC Core Knowledge
- A VPC is a virtual network dedicated to your AWS account
- Analogous to having your own data center inside AWS
- It is logically isolated from other virtual networks in the AWS Cloud
- Provides complete control over the virtual network environment including selection of IP ranges, creation of subnets, and configuration of route tables and gateways
- A VPC spans all AZs within a region
- When you create a VPC you must specify a range of IPv4 addresses for the VPC in the form of a CIDR block
- You have full control over who has access to the AWS resources inside your vpc
- A default VPC is created in each region with a public subnet in each AZ

## Defining VPC CIDR Blocks
- CIDR block size can be between /16 and /28
- The CIDR block must not overlap with any existing CIDR block that's associated with the VPC
- You cannot increase or decrease the size of an existing CIDR block
- The first four and last IP address are not available for use
- AWS recommend you use CIDR blocks from the RFC 1918 ranges:

| RFC 1918 ranges                                   | Example CIDR Block                                          |
|---------------------------------------------------|-------------------------------------------------------------|
| 10.0.0.0 - 10.255.255.255 (10/8 prefix)           | Your VPC must be /16 or smaller, for example, 10.0.0.0/16   |
| 172.16.0.0 - 172.31.255.255 (172.16/12 prefix)    | Your VPC must be /16 or smaller, for example, 172.31.0.0/16 |
| 192.168.0.0 - 192.168.255.255 (192.168/16 prefix) | Your VPC can be smaller, for example 192.168.0.0/20         |

### Additional Considerations
- Ensure you have enough networks and hosts
- Bigger CIDR blocks are typically better since they are more flexible
- Smaller subnets are OK for most use cases
- Consider deploying application tiers per subnet
- Split your HA resources across subnets in different AZs
- VPC Peering requires non-overlapping CIDR blocks, this is across all VPCs in all Regions/accounts you want to connect
- **Avoid overlapping CIDR blocks** as much as possible
