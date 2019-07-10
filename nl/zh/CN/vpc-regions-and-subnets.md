---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-07"

keywords: vpc, address prefix, region, subnet, zone, reserved, IP, ranges, deleting, creating, CIDR

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

# 了解 IP 地址范围、地址前缀、区域和子网
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (链接帮助主题)

本文档讨论了 {{site.data.keyword.cloud}} VPC 的区域、地址前缀和子网之间的关系：

* VPC 已部署并绑定到某个区域。
* 在该区域内，VPC 可以跨多个专区。
* 地址前缀支持在一个 VPC 所跨的各个专区之间进行通信。每个 VPC 都针对它所跨的每个专区获得一个快捷缺省地址前缀。
* 子网在地址前缀的作用域内创建，这意味着子网必须完全包含在单个现有地址前缀内。
* 如果将 [RFC 1918](https://tools.ietf.org/html/rfc1918) 定义的范围（`10.0.0.0/8`、`172.16.0.0/12` 或 `192.168.0.0/16`）之外的 IP 范围用于某个子网，那么连接到该子网的实例可能无法访问公用因特网的某些部分。

## IBM Cloud VPC 和区域
{: #ibm-cloud-vpc-and-regions}

{{site.data.keyword.cloud}} VPC 部署在一个区域中，但可能会跨多个专区。每个区域可包含多个专区，专区表示独立的故障区。

## IBM Cloud VPC 和地址前缀
{: #ibm-cloud-vpc-and-address-prefixes}

地址前缀支持在不同专区中的 {{site.data.keyword.cloud_notm}} VPC 实例之间进行通信。它们提供了_隐式路由器_需要的路由信息，用于将数据发送到目标实例所在的专区。每个子网必须包含在地址前缀中。{{site.data.keyword.cloud_notm}} VPC 不支持“本地”或“不可访问的”子网的概念。

_隐式路由器_是在 VPC 中创建的所有子网之间的固有网络连接。
{: note}

每个 {{site.data.keyword.cloud_notm}} VPC 最多可以有五个地址前缀用于每个专区。为了帮助您入门，{{site.data.keyword.cloud_notm}} VPC 定义了每个专区的缺省地址前缀（请参阅下表），但是，作为最佳实践，您应该在设计 VPC 寻址计划之后再部署该计划。

### VPC 缺省地址前缀
{: #default-vpc-address-prefixes}

创建新的 VPC 时，将根据区域和专区来分配缺省地址前缀，如下所示。

[Classic 访问 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) 具有不同的缺省地址前缀集。
{: important}

专区         | 地址前缀
---------------|---------------
`us-south-1`   | `10.240.0.0/18`
`us-south-2`   | `10.240.64.0/18`
`us-south-3`   | `10.240.128.0/18`
`eu-de-1`      | `10.243.0.0/18`
`eu-de-2`      | `10.243.64.0/18`
`eu-de-3`      | `10.243.128.0/18`
`jp-tok-1`     | `10.244.0.0/18`
`jp-tok-2`     | `10.244.64.0/18`
`jp-tok-3`     | `10.244.128.0/18`

将为新的专区或区域指定不同的缺省前缀。

### 地址前缀和 IBM Cloud 控制台 UI
{: #address-prefixes-and-the-ibm-cloud-console-ui}

使用 IBM Cloud 控制台 UI 创建 VPC 时，系统会自动选择地址前缀，并要求您在该缺省前缀中创建子网。如果此地址方案不符合您的需求，那么可以在创建 VPC 后定制地址前缀。然后，可以在定制的地址前缀中创建子网，并删除使用缺省前缀创建的子网。

需要此变通方法，才能通过 IBM Cloud 控制台 UI 使用 BYOIP。
{:note}

## IBM Cloud VPC 和子网
{: #ibm-cloud-vpc-and-subnets}

您可以将 {{site.data.keyword.cloud_notm}} VPC 划分为多个子网。{{site.data.keyword.cloud_notm}} VPC 中的所有子网都可以通过隐式路由器经专用 L3 路由到达另一个子网。您无需设置任何路由器或路径。

有关 VPC 中子网的有用事实：

* 子网由指定的 IP 地址范围组成。
* 子网绑定到单个专区，不能跨多个专区或区域。
* 子网可以跨 {{site.data.keyword.cloud_notm}} VPC 中的整个专区。
* 必须先创建 VPC，然后才能在该 VPC 中创建子网。
* IPv6 支持不可用。
* 可以将 VSI 关联到子网或解除 VSI 与子网的关联。（这需要添加 vNIC 并选择带宽。）
* 每个子网都必须包含在属于该子网所绑定的专区的地址前缀中。

您可以将自己的公共 IPv4 地址范围 (BYOIP) 添加到 {{site.data.keyword.cloud_notm}} VPC 帐户。使用 BYOIP 时，{{site.data.keyword.cloud_notm}} 必须在 {{site.data.keyword.cloud_notm}} 资源上配置这些 IPv4 地址，这将向提供的地址发送包以及接收来自这些地址的包。因此，在 {{site.data.keyword.cloud_notm}} 上使用提供的 IPv4 范围时，在您使用此服务的过程中，这些 IP 地址可能会向 IBM 的支持人员和第三方公开。
{:important}

![IBM Cloud VPC 概述](images/vpc-experience.svg "IBM Cloud VPC 概述")

### 使用子网的地址前缀
{: #using-address-prefixes-for-subnets}

每个子网必须存在于地址前缀中。
 * 对于新子网，可以从现有地址前缀中选取 IP 地址范围。
 * 如果专区的地址前缀不适合，那么可以编辑其中一个现有地址前缀，也可以为专区添加新的地址前缀（使用 API、CLI 或 UI 来添加）。

### 可用的 IP 地址
{: #available-ip-addresses}

下面是 **RFC 1918** 中定义的可用 IP 地址：

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

如果使用位于子网允许范围之外的 IP 范围（在先前部分中给定），那么连接到该子网的实例可能无法访问公用因特网的各部分。

### 有关创建子网的更多信息
{: #more-about-creating-a-subnet}

您可以通过以下两种方式为 VPC 指定子网：
  * 可以通过提供所需的子网大小（例如，支持的地址数，如 1024）来创建子网
  * 可以通过提供 CIDR 范围（例如，10.0.0.8/29）来创建子网

作为如何使用 CIDR 指定一个 1024 块的示例，如果指定的是 CIDR 范围而不是子网大小，那么 IPv4 块 `192.168.100.0/22` 表示从 `192.168.100.0` 到 `192.168.103.255` 的 1024 个 IPv4 地址。
{:tip}

