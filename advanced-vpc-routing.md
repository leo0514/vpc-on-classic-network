---

copyright:
  years: 2019
lastupdated: "2019-11-01"

keywords:

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
{:DomainName: data-hd-keyref="DomainName"}

# Setting up advanced routing
{: #setting-up-advanced-routing-in-vpc}

You can control the flow of network traffic by configuring VPC routes. Use routes to specify the next hop for packets, based on their destination addresses.
{:shortdesc}

Newer generation available. For more information, see [Setting up advanced routing in VPC](/docs/vpc?topic=vpc-advanced-routing) for generation 2 compute resources.
{:important}

## Routing structure

In the network structure of a VPC, one route table exists for each zone used by a VPC in a region. When an IP packet leaves a subnet, it is evaluated against the route table in the subnet's zone, to determine where to send the packet next. Each router has a default route, but the VPC Routes allow you to provide custom static routes to your routing tables.

## VPC route structure and behavior

A VPC route has three main components: the destination CIDR, the next hop, and the zone. The traffic that originates in a VPC in the zone whose destination address falls within the destination CIDR will be routed to the next hop. However, if the destination address falls within the destination CIDR for two routes, the most specific one will apply. Furthermore, if there are two or more, equally specific routes, the traffic will be round-robin distributed across each of the route's next hop.

## Managing VPC routes

You can manage your VPC routes using the user interface, CLI or API:
- Using the [IBM cloud console](https://{DomainName}/vpc/network/vpcs), go to the details page for a VPC and click **Routes** in the navigation.
- Using the CLI [ibmcloud is vpc-route-create](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-reference#vpc-route-create) command.
- Using the [routes API](https://{DomainName}/apidocs/vpc-on-classic#create-a-route-on-your-vpc).

## Limitations

1. Currently, the ability to set a custom default static route is not available.  Only non-default static routes are supported.
2. You can only set the `next_hop` to be an `ip_address`. Today, you cannot set the `next_hop` to be a VPN connection or a load balancer.
