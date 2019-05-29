---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: peering, classic, infrastructure, VRF, resources

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

# Classic peering for VPC
{: #setting-up-classic-peering-for-vpc}

{{site.data.keyword.cloud}} VPC is capable of peering with your classic IBM Cloud resources. Your accounts must meet a few requirements so that you can peer.

To get step by step instructions on how to create a VPC with classic peering enabled, see our [VPC Infrastructure document](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc) that covers this procedure with more detail.

## Requirements
{: #classic-peering-requirements}

1. Your VPC must be designated for "classic peering" at creation time, it cannot be designated at any later time.

2. The linked account to which you are peering your VPC must be VRF-enabled. For more information about VRF, please see [this explanatory document](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud).

3. You must also add a route back to your Classic-enabled VPC from each instance you are using on the Classic Infrastructure side.

## Restrictions
{: #classic-peering-restrictions}

Only 1 VPC per region can be enabled for classic peering.
