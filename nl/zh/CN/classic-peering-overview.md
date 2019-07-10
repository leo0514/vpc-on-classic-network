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

# VPC 的经典对等
{: #setting-up-classic-peering-for-vpc}

{{site.data.keyword.cloud}} VPC 能够与经典 IBM Cloud 资源建立对等连接。您的帐户必须满足一些要求才能建立对等连接。

要获取有关如何创建 VPC 并启用经典对等连接的逐步指示信息，请参阅我们的 [VPC 基础架构文档](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc)，其中更详细地讲述了此过程。

## 要求
{: #classic-peering-requirements}

1. 您的 VPC 必须在创建时指定用于“经典对等连接”，此后任何时间都无法再指定。

2. 您要与 VPC 建立对等连接的链接帐户必须已启用 VRF。有关 VRF 的更多信息，请参阅[本解释性文档](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud)。

3. 您还必须将路径从您在经典基础架构端使用的每个实例添加回经典支持的 VPC。

## 限制
{: #classic-peering-restrictions}

对于经典对等，每个区域只能启用 1 个 VPC。
