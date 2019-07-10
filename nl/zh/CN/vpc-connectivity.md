---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: capabilities, use cases, subnets, VPN, connections, reserved, IP, IPv4, floating

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

# 关于 VPC 的联网
{: #about-networking-for-vpc}

在本文档中，您将找到描述 {{site.data.keyword.cloud}} VPC 中用于子网、浮动 IP 地址和 VPN 连接的用例和功能的概念。

![IBM VPC Connectivity and Security](images/vpc-connectivity-and-security.svg "IBM VPC Connectivity and Security")

如下图所示：

* 子网可以通过可选的公共网关 (PGW) 连接到公用因特网。
* 可以将浮动 IP 地址 (FIP) 分配给任何 VSI，以便通过因特网访问 VSI，或通过 VSI 访问因特网。
* IBM Cloud VPC 中的子网提供专用连接；子网之间可以利用专用链接相互对话。您无需设置任何路径。
* 有关更多信息，请参阅[关于 VPC 基础架构](/docs/vpc-on-classic?topic=vpc-on-classic-about)。

## 术语
{: #terminology}

此[词汇表](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary)包含有关本文档中用于 IBM Cloud VPC 的术语的定义和信息。

## VPC 中子网的特征
{: #characteristics-of-subnets-in-the vpc}

子网由指定的 IP 地址范围（CIDR 块）组成。子网绑定到单个专区，无法跨多个专区或区域。但是，子网可以跨虚拟私有云内的专区抽象整体。同一 IBM Cloud VPC 中的子网可以相互连接。

### 专区 (Zones)
{: #zones}

专区是一个抽象概念，旨在帮助改进容错和缩短等待时间。专区可保证以下特性：

 * 每个专区都是一个独立的故障区，一个区域中的两个专区同时发生故障的可能性极低
 * 一个区域中各专区之间的流量的等待时间短于 2 毫秒

### 保留的 IP 地址
{: #reserved-ip-addresses}

运行虚拟私有云时，某些 IP 地址会保留供 IBM 使用。下面是保留的地址（给定的 IP 地址假定子网的 CIDR 范围为 10.10.10.0/24）：

  * CIDR 范围中的第一个地址 (10.10.10.0)：网络地址
  * CIDR 范围中的第二个地址 (10.10.10.1)：网关地址
  * CIDR 范围中的第三个地址 (10.10.10.2)：被 IBM 保留
  * CIDR 范围中的第四个地址 (10.10.10.3)：被 IBM 保留供未来使用
  * CIDR 范围中的最后一个地址 (10.10.10.255)：网络广播地址

### 使用公共网关建立子网外部连接
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

**公共网关 (PGW)** 支持子网（以及连接到该子网的所有 VSI）连接到因特网。请注意，缺省情况下，子网是专用的；但是，可以选择创建 PGW 并将子网连接到该 PGW。将子网连接到 PGW 后，该子网中的所有 VSI 都可以连接到因特网。

PGW 使用_多对一 NAT_，这意味着数千个使用专用地址的 VSI 将使用 1 个公共 IP 地址与公用因特网对话。PGW 不支持因特网发起与这些实例的连接。使用 API 可将子网连接到 PGW 以及从 PGW 拆离子网。

下图概述了网关服务的作用域。

![网关服务](images/scope-of-gateway-services.png)

### 使用浮动 IP 地址建立 VSI 外部连接
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

**浮动 IP 地址**是系统提供且可从公用因特网访问的 IP 地址。

您可以从 IBM 提供的可用浮动 IP 地址池中保留一个浮动 IP 地址，并且可以将该地址与同一虚拟私有云中的任何实例相关联或解除关联（借助于该实例的 vNIC）。任何浮动 IP 地址都可以关联到各种类型的虚拟服务器实例 (VSI)，例如负载均衡器或 VPN 网关。

浮动 IP 地址不能与多个接口相关联。必须在 VSI 上指定将与单个浮动 IP 关联的接口。该接口还将具有专用 IP 地址。后端系统在浮动 IP 与该接口的专用 IP 之间执行_一对一 NAT_ 操作。仅当特别请求释放操作时，才会释放浮动 IP。

**注：**
* **将浮动 IP 地址与 VSI 相关联会从前述 PGW 的多对一 NAT 中除去该 VSI。**
* **目前，浮动 IP 仅支持 IPv4 地址。**
* **您无法将自己的公共 IP 地址用作浮动 IP。**

有关 NAT 操作的更多信息，请参阅[相关因特网 RFC 文档 ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}。

### 使用 VPN 建立安全外部连接
{: #use-a-vpn-for-secure-external-connectivity}

虚拟专用网 (VPN) 服务可供用户通过因特网安全地连接到其 IBM Cloud VPC。有关分步骤指示信息，请参阅 [IBM 控制台 UI 指南](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)。

**VPN 功能**
  * 能够对 VPN 服务（站点到站点 IPSEC VPN）执行 CRUD（创建、读取、更新和删除）操作，以处理数据传输。
  * 能够将 VPN 服务关联到客户的 IBM Cloud VPC 或虚拟网络。
  * 能够通过静态定义或动态路由来识别客户的现场子网。
  * 支持安全密码，例如 SHA256、AES、3DES 和 IKEv2
  * 内置服务可靠性。
  * 监视 VPN 连接运行状况。
  * 支持发起者和响应者方式；也就是说，可以从隧道的任一侧发起流量。
