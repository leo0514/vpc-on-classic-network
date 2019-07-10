---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-29"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# IBM Cloud VPC 中的安全性
{: #security-in-your-ibm-cloud-vpc}

您可以使用安全组 (SG) 和/或网络访问控制表 (ACL) 这两种类型的控制措施来控制网络流量，以确保 VPC 和工作负载安全。请记住：

* 安全组按实例 (VSI) 控制流量。
* 访问控制表按子网控制流量。

## 安全性概述
{: #security-overview}

* 进出子网的流量可以由访问控制表 (ACL) 进行控制
* 安全组 (SG) 可以控制 VSI 级别的流量
* 设置供子网访问因特网的公共网关，该网关通过 ACL 保护
* 设置供 VSI 访问因特网的浮动 IP，该 IP 通过 SG 保护

![IBM VPC Connectivity and Security](images/vpc-connectivity-and-security.svg "IBM VPC Connectivity and Security")

## 定义
{: #definitions}

本[词汇表](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary)提供了 ACL 和 SG 的定义与描述以及可以对其执行的操作。以下部分描述了 ACL 和安全组的基本功能，以及 VPC 支持端到端加密的方式。

### 访问控制表 (Access Control List)
{: #access-control-list}

**访问控制表 (ACL)** 可以管理（即，可以允许或拒绝）子网的入站和出站流量。ACL 是无状态的，这意味着必须分别显式指定入站和出站规则。每个 ACL 都包含基于*源 IP*、*源端口*、*目标 IP*、*目标端口*和*协议*的规则。

在 {{site.data.keyword.cloud}} VPC 中，每个子网都是使用缺省 ACL 创建的，因此允许入站和出站流量，但客户可以创建定制 ACL。在任何时候，一个子网只能连接有一个 ACL，但一个 ACL 可以连接到多个子网。有关如何使用 ACL 的更多信息，请参阅 [ACL 指南](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)。

### 安全组 (Security Group)
{: #security-group}

**安全组**充当虚拟防火墙，用于控制一个或多个服务器 (VSI) 的流量。安全组是一组规则，用于指定是否允许关联 VSI 的流量。

客户创建 VSI 时，可以将一个或多个安全组与该 VSI 相关联。如果客户有正确的许可权，那么可以使用 IBM 控制台、CLI 或 API 来修改安全组规则。

有关如何创建使用安全组的 VSI 的更多信息，以及有关安全组功能的更多信息，请参阅[安全组指南](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups)。

### 端到端加密 (End-to-end encryption)
{: #end-to-end-encryption}

虽然 IBM Cloud VPC 未提供端到端加密，但支持此功能。例如，如果在虚拟服务器上有安全端点（例如，端口 443 上的 HTTPS 服务器），那么可以将浮动 IP 连接到该服务器，然后端口 443 上从客户机到服务器的连接会进行端到端加密。路径中的任何内容都不会导致解密。

但是，请注意，如果使用不安全的协议（例如，端口 80 上的 HTTP），那么数据将以明文形式在端到端之间流动。
