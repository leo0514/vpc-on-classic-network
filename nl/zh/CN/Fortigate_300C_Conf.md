---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, FortiGate, connection, secure

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 创建与远程 FortiGate 同级的安全连接
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

本文档基于 FortiGate 300C（固件版本 V5.2.13，build762 (GA)）。

以下示例步骤跳过了使用 {{site.data.keyword.cloud}} API 或 CLI 来创建虚拟私有云的先决条件步骤。有关更多信息，请参阅[入门](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)和[使用 API 设置 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 示例步骤
{: #fortigate-example-steps}

用于连接到远程 FortiGate 同级的拓扑类似于在两个 VPC 之间创建 VPN 连接。但是，一侧会替换为 FortiGate 300C 单元。

![在此输入图像描述](./images/vpc-vpn-fg-figure.png)

### 创建与远程 Fortigate 同级的安全连接
{: #to-create-a-secure-connection-with-the-remote-fortigate-peer}

转至 **VPN \> IPsec \> 隧道**，然后创建新的定制隧道或编辑现有隧道。

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

FortiGate 单元收到来自远程 VPN 同级的连接请求时，会使用 IPsec 第 1 阶段参数来建立安全连接，并对该 VPN 同级进行认证。随后，如果安全策略允许该连接，FortiGate 单元将使用 IPsec 阶段参数来建立隧道，并应用 IPsec 安全策略。密钥管理、认证和安全服务均通过 IKE 协议动态协商。

**为了支持这些功能，FortiGate 单元必须执行以下常规配置步骤：**

* 定义 FortiGate 单元对远程同级进行认证并建立安全连接所需的第 1 阶段参数。

* 定义 FortiGate 单元创建与远程同级的 VPN 隧道所需的第 2 阶段参数。

* 创建安全策略，用于控制 IP 源地址与目标地址之间允许的服务和允许的流量方向。

缺省情况下，已设置一些典型的第 1 阶段和第 2 阶段参数。

### 针对 IBM Cloud VPC VPN 进行配置
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

要连接到 IBM Cloud VPC 的 VPN 功能，建议使用以下配置：

1. 选择 IKEv2 作为认证方式；
2. 在第 1 阶段建议中启用 `DH-group 2`
3. 在第 1 阶段建议中设置 `lifetime = 36000`
4. 在第 2 阶段建议中禁用 PFS
5. 在第 2 阶段建议中设置 `lifetime = 10800`
6. 在第 2 阶段输入子网的信息
7. 其余参数保留其缺省值。

![在此输入图像描述](./images/vpc-vpn-fg-network.JPG)

### 网络参数
{: #fortigate-network-parameters}

![在此输入图像描述](./images/vpc-vpn-fg-authentication.JPG)

### 认证参数
{: #fortigate-authentication-parameters}

![在此输入图像描述](./images/vpc-vpn-fg-phase1.JPG)

### 第 1 阶段参数
{: #fortigate-phase-1-parameters}

![在此输入图像描述](./images/vpc-vpn-fg-phase2.JPG)

### 第 2 阶段参数
{: #fortigate-phase-2-parameters}

## 创建与本地 IBM Cloud VPC 的安全连接
{: #fortigate-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

要创建安全连接，需要在 VPC 中创建 VPN 连接，这类似于 2 个 VPC 的示例。

* 在 VPC 子网上创建 VPN 网关，并创建 VPC 与 FortiGate 单元之间的 VPN 连接，将 `local_cidrs` 设置为 VPC 上的子网值，将 `peer_cidrs` 设置为 FortiGate 上的子网值。

**注：**创建 VPN 网关时，网关状态显示为 `pending`，创建完成后，状态会变为 `available`。创建过程可能需要一些时间。

![在此输入图像描述](images/vpc-vpn-fg-connection.png)

### 检查安全连接的状态
{: #fortigate-check-the-status-of-the-secure-connection}

可以通过 IBM Cloud 控制台来检查连接的状态。此外，还可以尝试使用 VSI 执行站点到站点的 `ping` 操作。

![在此输入图像描述](images/vpc-vpn-fg-status.JPG)
