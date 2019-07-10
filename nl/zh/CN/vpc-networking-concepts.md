---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: VRF, router, hypervisor, address prefixes, classic access, implicit router, packet flows, NAT, data flows

subcollection: vpc-on-classic-network


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}

# VPC 后台操作
{: #vpc-behind-the-curtain}

本页面提供了 VPC 联网“后台”操作的更详细的概念性概述。读者应该具备一些联网背景知识。

## 网络隔离
{: #network-isolation}

VPC 网络隔离在三个级别执行：

* **系统管理程序**：VSI（虚拟服务器实例）通过系统管理程序本身隔离。一个 VSI 无法直接访问由同一个系统管理程序托管，但不在同一个 VPC 中的其他 VSI。

* **网络**：通过使用**虚拟网络标识** (VNI) 在网络级别执行隔离。这些标识的作用域限定为本地专区。这些 VNI 会添加到进入 VPC 的任何专区的所有数据包中：在由 VSI 发送时从系统管理程序进入，或者在由隐式路由功能发送时从云中进入专区。

离开专区的包会除去 VNI。当包通过隐式路由功能进入而到达其目标专区时，隐式路由器会始终为该专区添加正确的 VNI。
{: note}

* **路由器**：_隐式路由器功能_通过在云主干中提供**虚拟路由功能** (VRF) 和使用 MPLS（多协议标签交换）的 VPN，实现对每个 VPC 的隔离。每个 VPC 的 VRF 都具有唯一标识，此隔离允许每个 VPC 有权访问其自己的 IPv4 地址空间副本。MPLS VPN 支持联合云的所有边缘：经典基础架构、Direct Link 和 VPC。

## 地址前缀
{: #address-prefixes}

地址前缀是 VPC 的隐式路由功能用于查找_目标 VSI_ 的摘要信息，与目标 VSI 所在的可用性专区无关。地址前缀的主要功能是优化通过 MPLS VPN 进行的路由，同时避免病态路由情况。在 VPC 中创建的所有子网都必须包含在地址前缀中，以便该 VPC 中的所有 VSI 都可相互访问。

## 数据包流和隐式路由器
{: #data-packet-flows-and-the-implicit-router}

在 VPC 中执行的 VSI 数据包流有 6 种不同的类型。下面是这 6 种流，按复杂性升序列出：

* 子网内，主机内（同一系统管理程序）
* 子网内，主机间
* 子网间，专区内
* 子网间，专区间
* 额外 VPC 服务（对于 IaaS 或 CSE 访问）
* 额外 VPC 因特网（对于因特网访问）

**子网内、主机内**数据流：这些是最简单的数据流。包在系统管理程序上的 VSI 之间流动，并且没有任何包离开系统管理程序。

**子网内、主机间**数据流：这些数据流涉及离开系统管理程序的包。每个包都会使用正确的 VNI（虚拟化网络标识）进行标记，以确保数据隔离，然后会将包发送到托管目标 VSI 的目标系统管理程序。目标系统管理程序会除去数据包的 VNI，然后将数据包转发到目标 VSI。

**子网间、专区内**数据流：这些数据流涉及利用 VPC 隐式路由器功能的包，该功能将连接 VPC 中创建的所有子网。它会将数据包路由到正确的目标系统管理程序。如果目标系统管理程序与源系统管理程序不同，那么会使用正确的 VNI 来标记数据包，并将其发送到目标系统管理程序。在目标系统管理程序中，会除去数据包的 VNI，然后将数据包转发到目标 VSI。（这些最后的步骤与上一个类型数据流中所述的相同。)

**子网间、专区间**数据流：对于这些数据流，隐式路由器功能会除去 VNI 并将包转发到 VPC 的 MPLS VPN 中，以便在云主干中传输。在目标专区中，隐式路由器功能会使用相应的 VNI 标记数据包。然后，该包会转发到目标系统管理程序，其中会除去数据包的 VNI（如前所述），以便可以将数据包转发到目标 VSI。

**额外 VPC 服务**数据流：发往 IaaS 或 IBM Cloud 服务端点 (CSE) 服务的包将利用 VPC 的隐式路由器功能以及网络地址转换 (NAT) 功能。该转换功能会将 VSI 地址替换为 IPv4 地址，以用于向所请求的 IaaS 或 CSE 服务标识该 VPC。

**额外 VPC 因特网**数据流：发往因特网的包最复杂。除了利用 VPC 的隐式路由器功能外，每个这样的流还依赖于隐式路由器的两个网络地址转换 (NAT) 功能之一：

  * 显式一对多 NAT，通过服务于所连接全部子网的公共网关功能提供。
  * 分配给单个 VSI 的一对一 NAT。

NAT 转换后，隐式路由器会使用云主干将这些发往因特网的包转发到因特网。

## 经典访问
{: #classic-access}

VPC 的[**经典访问**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc)功能的实现方法是将 {{site.data.keyword.cloud}} 经典基础架构帐户中的 VRF 标识复用为 VPC 的 VRF 标识。此实现允许 VPC 的隐式路由器功能加入经典基础架构帐户使用的 MPLS VPN。因此，VPC 有权访问经典资源，也有权访问通过现有 Direct Link 连接可访问的其他任何内容。
