---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Vyatta, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 创建与远程 Vyatta 同级的安全连接
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

本文档基于 Vyatta 版本：AT&T vRouter 5600 1801d。

以下示例步骤跳过了使用 {{site.data.keyword.cloud}} API 或 CLI 来创建虚拟私有云的先决条件步骤。有关更多信息，请参阅[入门](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)和[使用 API 设置 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 示例步骤
{: #vyatta-example-steps}

用于连接到远程 Vyatta 同级的拓扑类似于在两个 {{site.data.keyword.cloud_notm}} VPC 之间创建 VPN 连接。但是，一侧会替换为 Vyatta 单元。

![在此输入图像描述](images/vpc-vpn-vy-figure.png)

### 创建与远程 Vyatta 同级的安全连接
{: #vyatta-to-create-a-secure-connection-with-the-remote-vyatta-peer}

VPN 同级收到来自远程 VPN 同级的连接请求时，会使用 IPsec 第 1 阶段参数来建立安全连接，并对该 VPN 同级进行认证。随后，如果安全策略允许连接，Vyatta 单元将使用 IPsec 第 2 阶段参数来建立隧道，并应用 IPsec 安全策略。密钥管理、认证和安全服务均通过 IKE 协议动态协商。

可以转至 Vyatta 命令行来配置 IPsec 隧道，也可以上传 `*.vcli` 文件来装入配置。

**为了支持这些功能，Vyatta 单元必须执行以下常规配置步骤：**

* 定义 Vyatta 单元对远程同级进行认证并建立安全连接所需的第 1 阶段参数。

* 定义 Vyatta 单元创建与远程同级的 VPN 隧道所需的第 2 阶段参数。

要连接到 IBM Cloud VPC 的 VPN 功能，建议使用以下配置：

1. 选择 `IKEv2` 作为认证方式；
2. 在第 1 阶段建议中启用 `DH-group 2`
3. 在第 1 阶段建议中设置 `lifetime = 36000`
4. 在第 2 阶段建议中禁用 PFS
5. 在第 2 阶段建议中设置 `lifetime = 10800`
6. 在第 2 阶段中输入同级和子网的信息

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.161.151_test_ike
set security vpn ipsec ike-group 169.61.161.151_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.161.151_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.161.151_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.161.151_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.161.151_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.161.151_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.161.151 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.161.151 authentication pre-shared-secret ******
set security vpn ipsec site-to-site peer 169.61.161.151 ike-group 169.61.161.151_test_ike
set security vpn ipsec site-to-site peer 169.61.161.151 default-esp-group 169.61.161.151_test_ipsec
set security vpn ipsec site-to-site peer 169.61.161.151 description "automation test"
set security vpn ipsec site-to-site peer 169.61.161.151 local-address 169.45.74.119
set security vpn ipsec site-to-site peer 169.61.161.151 connection-type initiate


set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 local prefix 192.168.200.0/24
set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 remote prefix 192.168.17.0/28

commit
```
{: screen}

然后，可以使用 SCP 将此 `*.vcli` 文件上传到 Vyatta 以应用配置。

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### 创建与本地 IBM Cloud VPC 的安全连接
{: #vyatta-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

 要创建安全连接，需要在 VPC 中创建 VPN 连接，这类似于 2 个 VPC 的示例。

* 在 VPC 子网上创建 VPN 网关，并创建 VPC 与 Vyatta 单元之间的 VPN 连接，将 `local_cidrs` 设置为 VPC 上的子网值，将 `peer_cidrs` 设置为 Vyatta 单元上的子网值。

创建 VPN 网关时，网关状态显示为 `pending`，创建完成后，状态会变为 `available`。创建过程可能需要一些时间。
{: note}

![在此输入图像描述](images/vpc-vpn-vy-connection.png)

### 检查安全连接的状态
{: #vyatta-check-the-status-of-the-secure-connection}

您可以通过 {{site.data.keyword.cloud_notm}} 控制台来检查连接的状态。此外，还可以尝试使用 VSI 执行站点到站点的 `ping` 操作。

![在此输入图像描述](images/vpc-vpn-vy-status.png)
