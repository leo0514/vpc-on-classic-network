---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-12-05"

keywords: vpc, vpc network, secure, region, zone, subnet, terminology, public gateway, floating IP, NAT

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}
{:external: target="_blank" .external}

# About networking for VPC
{: #about-networking-for-vpc}

A Virtual Private Cloud (VPC) is a virtual network that's tied to your customer account. It gives you cloud security, with the ability to scale dynamically, by providing fine-grained control over your virtual infrastructure and your network traffic segmentation.

This document covers some networking concepts as they are applied within {{site.data.keyword.vpc_full}}. The use cases and characteristics described include:

* regions
* zones
* reserved IP addresses
* public gateways
* vNIC interfaces
* subnets
* floating IP addresses
* VPN connections

## Overview
{: #subnets-overview}

Three options are available for access to the Internet from your VPC:
* Use a public gateway for Internet traffic from the whole subnet.
* Use a Floating IP for Internet traffic from and to an instance.
* Use a VPN for secure external connectivity.

Remember:
* Some IP ranges are reserved by IBM.
* You must create your VPC before you create subnets within that VPC.
* IPv6 support is not available.
* Secondary IP addresses on a single network interface are not supported.

Optionally, you can create a [Classic Access VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc) to connect to your IBM Cloud Classic Infrastructure.
{:note}

As shown in the following figure:
* Subnets in your VPC can connect to the public Internet through an optional Public Gateway (PGW).
* You can assign a Floating IP address (FIP) to any instance to reach it from the Internet, or vice versa, independent of whether the subnet is attached to a public gateway.
* Subnets within the {{site.data.keyword.vpc_short}} offer private connectivity; they can talk to each other over a private link, through the implicit router. Setting up routes is not necessary.

![{{site.data.keyword.vpc_short}} Connectivity and Security](images/vpc-connectivity-and-security.svg "IBM VPC Connectivity and Security"){: caption="Figure: You can subdivide a Virtual Private Cloud with subnets, and each subnet can reach the public Internet, if desired." caption-side="top"}

## Terminology
{: #network-terminology}

When working with your VPC, you'll need to be familiar with the basic concepts of _region_ and _zone_ as they apply to your deployment.

### Regions
{: #subnet-regions}

A region is an abstraction related to the geographic area in which a VPC is deployed. Each region contains multiple zones, which represent independent fault domains. A VPC may span multiple zones within its assigned region.

### Zones
{: #subnet-zones}

A zone is an abstraction that refers to the physical data center that hosts the compute, network, and storage resources, as well as the related cooling and power, which provides services and applications. Isolation of zones improves the system's overall fault tolerance, decreases latency, and avoids creating a single shared point of failure. A zone guarantees the following properties:

 * Each zone is an independent fault domain, and it is extremely unlikely for two zones in a region to fail simultaneously.
 * Traffic between zones in a region will have less than 2ms of latency.

## Characteristics of subnets in the VPC
{: #characteristics-of-subnets}

A subnet consists of a specified IP address range (CIDR block). Subnets are bound to a single zone, and they cannot span multiple zones or regions. However, a subnet can span the entirety of the zone abstractions within their Virtual Private Cloud. Subnets in the same VPC are connected to each other.

### Reserved IP addresses
{: #reserved-ip-addresses}

Certain IP addresses are reserved for use by IBM when operating the Virtual Private Cloud. Here are the reserved addresses (these IP addresses assume that the subnet's CIDR block is 10.10.10.0/24):

  * First address in the CIDR block (10.10.10.0): Network address
  * Second address in the CIDR block (10.10.10.1): Gateway address
  * Third address in the CIDR block (10.10.10.2): reserved by IBM
  * Fourth address in the CIDR block (10.10.10.3): reserved by IBM for future use
  * Last address in the CIDR block (10.10.10.255): Network broadcast address

### Use a public gateway for external connectivity of a subnet
{: #use-a-public-gateway}

A **public gateway (PGW)** enables a subnet (with all the instances attached to the subnet) to connect to the Internet. Note that subnets are private by default; however, optionally, you can create a PGW and attach a subnet to the PGW. After a subnet is attached to the PGW, all the instances in that subnet can connect to the Internet.

PGW uses _Many-to-1 NAT_, which means that thousands of instances with private addresses will use 1 public IP address to talk to the public Internet. PGW does not enable the Internet to initiate a connection with those instances. Use the API to attach and detach subnets to and from your PGW.

The following figure summarizes the current scope of gateway services.

| SNAT | DNAT | ACL | VPN |
| ---- | ---- | --- | --- |
| Instances can have outbound-only access to the Internet | Allow inbound connectivity from the Internet to a Private IP | Provide restricted inbound access from the Internet to instances or subnets | Site-to-Site VPN handles customers of any size, and single or multiple locations |
| Entire subnets share the same outbound public endpoint | Provides limited access to a single private server | Restrict access inbound from Internet, based on service, protocol, or port | High throughput (up to 10 Gbps) provides customers the ability to transfer large data files securely and quickly |
| Protects instances; Cannot initiate access to instances through the public endpoint | DNAT service can be scaled up or down, based on requirements | Stateless ACLs allow for granular control of traffic | Create secure connections with industry standard encryption |

A public gateway is created in a VPC, but the gateway does nothing until it is attached to a subnet. You can create only one public gateway per zone, which means, for example, that you could have three (3) public gateways per VPC in an environment with 3 zones.
{:note}

## Limitations of subnets
{: #limitations-of-subnets}

For a complete list of known limitations and features not currently supported, refer to the [Known Limitations](/docs/vpc-on-classic?topic=vpc-on-classic-known-limitations).

### Restrictions on deleting a subnet
{: #restrictions-on-deleting-a-subnet}

You cannot delete a subnet if resources (such as a Virtual Server Instances (VSIs), or floating IPs) are in use in that subnet, the resources must be deleted first.

### Limitations on updating an existing subnet
{: #limitations-on-updating-an-existing-subnet}

* You cannot resize an existing subnet. For example, 10.10.16.0/24 cannot be resized to 10.10.16.0/20.
* You cannot move an existing subnet. For example, 10.10.10.0/24 cannot be moved to 10.10.11.0/24.

## External connectivity
{: #external-connectivity}

External connectivity can be achieved by means of a floating IP address attached to an instance, by a public gateway attached to a subnet, or by a VPN tunnel.

### Use a Floating IP address for external connectivity of an instance 
{: #use-floating-ip}

**Floating IP addresses** are IP addresses that are provided by {{site.data.keyword.cloud_notm}} and are reachable from the public Internet.

You can reserve a Floating IP address from the pool of available Floating IP addresses provided by IBM, and you can associate or disassociate it with any instance in a VPC, by means of the vNIC for that instance. A Floating IP address can be associated to a virtual server instance (VSI), load balancer, or VPN gateway.

Your Floating IP address cannot be associated with multiple interfaces. You must specify the interface on the VSI that will be associated with that individual Floating IP. That interface also will have a private IP address. The backend system performs _1-to-1 NAT_ operations between the Floating IP and the Private IP of that interface.

There are costs associated with a Floating IP even when the Floating IP is not associated. You must release the Floating IP to stop incurring charges. After a Floating IP is released, it will be returned to the pool of IBM Cloud for potential reallocation. 
{: important}

**Important things to note:**
* **Associating a Floating IP address with a VSI removes the VSI from the PGW's Many-to-1 NAT mentioned previously.**
* **Currently, Floating IP supports only IPv4 addresses.**
* **You cannot bring your own public IP address to use as a floating IP.**

For more information about NAT operations, refer to [the related Internet RFC document](http://www.faqs.org/rfcs/rfc1631.html){: external}.

### Use VPN for secure external connectivity
{: #use-vpn}

Virtual Private Network (VPN) service is available for users to connect to their {{site.data.keyword.vpc_short}} from the Internet, securely.

**VPN capabilities**
  * Ability to associate a VPN service to a {{site.data.keyword.vpc_short}}.
  * Ability to CRUD (Create, Read, Update and Delete) VPN service (site-to-site IPsec VPN) for handling data transfer.
  * Uses static routes to identify on-site subnets.
  * Support for secure ciphers such as SHA256, AES, 3DES, IKEv2.
  * Built-in service reliability and high availability.
  * Continuous monitoring of VPN connection health.
  * Support for both initiator and responder modes; that is, traffic may be initiated from either side of the tunnel.

## Learn more
{: #subnets-learn-more}
   * [IBM VPC security](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-security-in-your-ibm-cloud-vpc)
   * [IBM VPC addresses, regions, and subnets](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
