---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: capabilities, use cases, subnets, VPN, connections, reserved, IP, IPv4, floating, vpc, vpc network

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}
{:external: target="_blank" .external}

# About Networking for VPC
{: #about-networking-for-vpc}

In this document, you'll find concepts that describe use cases and capabilities within {{site.data.keyword.cloud}} VPC for subnets, Floating IP addresses, and VPN connections.

![IBM VPC Connectivity and Security](images/vpc-connectivity-and-security.svg "IBM VPC Connectivity and Security"){: caption="Figure: You can subdivide a Virtual Private Cloud with subnets, and each subnet can reach the public Internet, if desired." caption-side="top"}

As shown in the figure:

* Subnets can connect to the public internet through an optional Public Gateway (PGW).
* You can assign a Floating IP address (FIP) to any VSI to reach it from the internet, or vice versa.
* Subnets in the IBM Cloud VPC offer private connectivity; they can talk to each other over a private link. You do not need to set up any routes.
* For more information, see our [About VPC Infrastructure](/docs/vpc-on-classic?topic=vpc-on-classic-about).

## Terminology
{: #terminology}

This [glossary](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) contains definitions and information about terms used in this document for IBM Cloud VPC.

## Characteristics of subnets in the VPC
{: #characteristics-of-subnets-in-the vpc}

A subnet consists of a specified IP address range (CIDR block). Subnets are bound to a single zone, and they cannot span multiple zones or regions. However, a subnet can span the entirety of the Zone abstractions within their Virtual Private Cloud. Subnets in the same IBM Cloud VPC are connected to each other.

### Zones
{: #zones}

A Zone is an abstraction designed to assist with improved fault tolerance and decreased latency. A zone guarantees the following properties:

 * Each zone is an independent fault domain and it is extremely unlikely for two zones in a region to fail simultaneously
 * Traffic between zones in a region will be < 2ms latency

### Reserved IP Addresses
{: #reserved-ip-addresses}

Certain IP addresses are reserved for use by IBM when operating the Virtual Private Cloud. Here are the reserved addresses (the given IP addresses assume that the subnet's CIDR range is 10.10.10.0/24):

  * First address in the CIDR range (10.10.10.0): Network address
  * Second address in the CIDR range (10.10.10.1): Gateway address
  * Third address in the CIDR range (10.10.10.2): reserved by IBM
  * Fourth address in the CIDR range (10.10.10.3): reserved by IBM for future use
  * Last address in the CIDR range (10.10.10.255): Network broadcast address

### Use a Public Gateway for external connectivity of a subnet
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

A **Public Gateway (PGW)** enables a subnet (with all the VSIs attached to the subnet) to connect to the internet. Note that subnets are private by default; however, optionally, you can create a PGW and attach a subnet to the PGW. After a subnet is attached to the PGW, all the VSIs in that subnet can connect to the internet.

PGW uses _Many-to-1 NAT_, which means that thousands of VSIs with private addresses will use 1 public IP address to talk to the public internet. PGW does not enable the internet to initiate a connection with those instances. Use the API to attach and detach subnets to and from your PGW.

The following figure summarizes the scope of gateway services.

![gateway services](images/scope-of-gateway-services.png)

### Use a Floating IP address for external connectivity of a VSI
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

**Floating IP addresses** are IP addresses that are provided by the system and are reachable from the public internet.

You can reserve a Floating IP address from the pool of available Floating IP addresses provided by IBM, and you can associate or disassociate it with any instance in the same Virtual Private Cloud, by means of the vNIC for that instance. Any Floating IP address can be associated to various types of virtual server instances (VSIs), for example, a load balancer or a VPN gateway.

Your Floating IP address cannot be associated with multiple interfaces. You must specify the interface on the VSI that will be associated with that individual Floating IP. That interface also will have a private IP address. The backend system performs _1-to-1 NAT_ operations between the Floating IP and the private IP of that interface. A Floating IP is released only when you specifically request the release action. 

**Notes:**
* **Associating a Floating IP address with a VSI removes the VSI from the PGW's Many-to-1 NAT mentioned previously.**
* **Currently, Floating IP supports only IPv4 addresses.**
* **You cannot bring your own public IP address to use as a Floating IP.**

For more information about NAT operations, please refer to [the related Internet RFC document](http://www.faqs.org/rfcs/rfc1631.html){: external}.

### Use a VPN for secure external connectivity
{: #use-a-vpn-for-secure-external-connectivity}

Virtual Private Network (VPN) service is available for users to connect to their IBM Cloud VPC from the internet, securely. For step-by-step instructions, please see our [IBM Console UI guide](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console).

**VPN Capabilities**
  * Ability to CRUD (Create, Read, Update and Delete) VPN service (site-to-site IPSEC VPN) for handling data transfer.
  * Ability to associate VPN service to a customer's IBM Cloud VPC or virtual network.
  * Ability to identify customer's on-site subnets either by static definition or by dynamic routing.
  * Support for secure ciphers such as SHA256, AES, 3DES, IKEv2
  * Built-in service reliability.
  * Monitoring of VPN connection health.
  * Support for both initiator and responder modes; that is, traffic may be initiated from either side of the tunnel.
