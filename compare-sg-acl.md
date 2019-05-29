---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# Comparing Security Groups and Access Control Lists
{: #compare-security-groups-and-access-control-lists}

Security groups and access control lists provide ways to control the traffic across the subnets and instances in your {{site.data.keyword.cloud}} VPC, using rules that you specify.

The following table summarizes some key differences between security groups and ACLs:

|  | Security Groups | ACLs    |
|-------------|-----------------|---------|
| Control level  | VSI instance    | Subnet  |
| State   | Stateful - Once an inbound connection is permitted, it is allowed to reply | Stateless - Both inbound and outbound connections must be explicitly allowed |
| Supported rules | Uses allow rules only | Uses allow and deny rules|
| How rules are applied | All rules are considered | Rules are processed in sequence |
| Relationship to the associated resource | An instance can be associated with multiple security groups| Multiple subnets can be associated with the same ACL|
