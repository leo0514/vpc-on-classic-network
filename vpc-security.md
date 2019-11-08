---

copyright:
  years: 2018, 2019
lastupdated: "2019-11-01"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption, vpc, vpc network

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Security in your IBM Cloud VPC
{: #security-in-your-ibm-cloud-vpc}

You can keep your VPC and workloads secure by controlling network traffic using security groups (SGs), by using network access control lists (ACLs), or by using both types of control. Remember:

* Security groups control traffic on a per-instance (VSI) basis.
* Access control lists control traffic on a per-subnet basis.

## Security overview
{: #security-overview}

* Traffic to and from a subnet can be controlled by Access Control Lists (ACLs)
* Security Groups (SGs) can control the traffic at the VSI level
* Set up a public gateway for subnet access to the Internet, guarded by ACLs
* Set up a floating IP for VSI access to the Internet, guarded by SGs

![IBM VPC Connectivity and Security](images/vpc-connectivity-and-security.svg "IBM VPC Connectivity and Security"){: caption="Figure: Security groups and ACLs add security to your subnets and instances." caption-side="top"}

## Definitions
{: #definitions}

The section that follows describes basic functionality of ACLs and security groups, and the ways in which VPC supports end-to-end encryption.

### Access Control List
{: #access-control-list}

An **Access Control List (ACL)** can manage (that is, it can allow or deny) inbound and outbound traffic for a subnet. An ACL is stateless, which means that inbound and outbound rules must be specified separately and explicitly. Each ACL consists of rules, based upon a *source IP*, *source port*, *destination IP*, *destination port*, and *protocol*.

In {{site.data.keyword.cloud}} VPC, every subnet is created with a default ACL, which allows inbound and outbound traffic, but customers can create custom ACLs. Only one ACL is attached to a subnet at any time, but one ACL can be attached to multiple subnets. For more information about how to use ACLs, see our [ACL guide](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls).

### Security Group
{: #security-group}

A **security group** acts as a virtual firewall that controls the traffic for one or more servers (VSIs). A security group is a collection of rules that specify whether to allow traffic for an associated VSI. 

When a customer creates a VSI, he or she can associate one or more security groups with that VSI. Given the correct permissions, customers can modify security group rules using the IBM Console, the CLI, or the API.

For more information about how to create a VSI that uses security groups, and more about the capabilities of security groups, refer to our [security groups guide](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups).

### End-to-end encryption
{: #end-to-end-encryption}

Although IBM Cloud VPC does not provide end-to-end encryption, it allows for it. For example, if you have a secure endpoint on a virtual server (for example, an HTTPS server on Port 443), you can attach a floating IP to that server, and then your connection is end-to-end encrypted from the client to the server on Port 443.  Nothing in the path forces a decryption.

Note, however, that if you use an insecure protocol such as HTTP on Port 80, your data is in the clear from end to end.
