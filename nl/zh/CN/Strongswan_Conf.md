---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{: note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 创建与远程 StrongSwan 同级的安全连接
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

本文档基于 Strongswan 版本：Linux StrongSwan U5.3.5/K4.4.0-133-generic。

以下示例步骤跳过了使用 {{site.data.keyword.cloud}} API 或 CLI 来创建虚拟私有云的先决条件步骤。有关更多信息，请参阅[入门](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)和[使用 API 设置 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 示例步骤
{: #strongswan-example-steps}

用于连接到远程 StrongSwan 同级的拓扑类似于在两个 VPC 之间创建 VPN 连接。但是，连接的一侧会替换为 strongSwan 单元。

![在此输入图像描述](./images/vpc-vpn-sw-figure.png)

### 创建与远程 StrongSwan 同级的安全连接
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

转至 **/etc** 并创建新的定制隧道配置文件，名称类似于 **ipsec.abc.conf**。编辑 **/etc/ipsec.conf** 并通过添加以下行来包含 **ipsec.abc.conf**：

    include /etc/ipsec.abc.conf

VPN 同级收到来自远程 VPN 同级的连接请求时，会使用 IPsec 第 1 阶段参数来建立安全连接，并对该 VPN 同级进行认证。随后，如果安全策略允许连接，StrongSwan 单元将使用 IPsec 第 2 阶段参数来建立隧道，并应用 IPsec 安全策略。密钥管理、认证和安全服务均通过 IKE 协议动态协商。

**为了支持这些功能，StrongSwan 单元必须执行以下常规配置步骤：**

* 定义 StrongSwan 对远程同级进行认证并建立安全连接所需的第 1 阶段参数。

* 定义 StrongSwan 创建与远程同级的 VPN 隧道所需的第 2 阶段参数。要连接到 IBM Cloud VPC 的 VPN 功能，建议使用以下配置：

1. 选择 `IKEv2` 作为认证方式；
2. 在第 1 阶段建议中启用 `DH-group 2`
3. 在第 1 阶段建议中设置 `lifetime = 36000`
4. 在第 2 阶段建议中禁用 PFS
5. 在第 2 阶段建议中设置 `lifetime = 10800`
6. 在第 2 阶段建议中输入同级和子网的信息

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: screen}

在 `/etc/ipsec.secrets` 中设置预共享密钥

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

在配置文件执行完毕后，重新启动 StrongSwan 单元。

```
 ipsec restart
```
{: screen}

### 创建与本地 IBM Cloud VPC 的安全连接
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* 要创建安全连接，需要在 VPC 中创建 VPN 连接，这类似于 2 个 VPC 的示例。

* 在 VPC 子网上创建 VPN 网关，并创建 VPC 与 StrongSwan 之间的 VPN 连接，将 `local_cidrs` 设置为 VPC 上的子网值，将 `peer_cidrs` 设置为 StrongSwan 上的子网值。

创建 VPN 网关时，网关状态显示为 `pending`，创建完成后，状态会变为 `available`。创建过程可能需要一些时间。
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### 检查安全连接的状态
{: #strongswan-check-the-status-for-a-secure-connection}

可以通过 IBM Cloud 控制台来检查连接的状态。此外，还可以尝试使用 VSI 执行站点到站点的 `ping` 操作。

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
