---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# 创建与远程 Cisco ASAv 同级的安全连接
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

本文档基于 Cisco ASAv：Cisco Adaptive Security Appliance V9.10(1) 软件。

以下示例步骤跳过了使用 {{site.data.keyword.cloud}} API 或 CLI 来创建虚拟私有云的先决条件步骤。有关更多信息，请参阅[入门](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)和[使用 API 设置 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 示例步骤
{: #cisco-example-steps}

用于连接到远程 Cisco ASAv 同级的拓扑类似于在两个 {{site.data.keyword.cloud}} 虚拟私有云之间创建 VPN 连接。但是，一侧会替换为 Cisco ASAv 单元。

![在此输入图像描述](./images/vpc-vpn-asav-figure.png)

### 创建与远程 Cisco ASAv 同级的安全连接
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

配置 Cisco ASAv 以用于 IBM VPC VPN 的第一步是确保设置了以下先决条件：

* Cisco ASAv 已使用正确的许可证联机并正常运行
* 已启用 Cisco ASAv 的密码
* 至少有一个已配置且已验证的功能性内部接口
* 至少有一个已配置且已验证的功能性外部接口

Cisco ASAv 单元收到来自远程 VPN 同级的连接请求时，会使用 IPsec 第 1 阶段参数来建立安全连接，并对该 VPN 同级进行认证。随后，如果安全策略允许连接，Cisco ASAv 将使用 IPsec 第 2 阶段参数来建立隧道，并应用 IPsec 安全策略。密钥管理、认证和安全服务均通过 IKE 协议动态协商。

**为了支持这些功能，Cisco ASAv 单元必须执行以下常规配置步骤：**

* 定义 Cisco ASAv 单元对远程同级进行认证并建立安全连接所需的第 1 阶段参数。
* 定义 Cisco ASAv 单元创建与远程同级的 VPN 隧道所需的第 2 阶段参数。

创建因特网密钥交换 (IKE) V2 建议对象。IKEv2 建议对象包含定义远程访问和站点到站点 VPN 策略时创建 IKEv2 建议所需的参数。IKE 是一种密钥管理协议，有助于管理基于 IPsec 的通信。IKE 用于认证 IPsec 同级，协商和分发 IPsec 加密密钥，以及自动建立 IPsec 安全性关联 (SA)。 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2 
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <key value>
 ikev2 remote-authentication pre-shared-key <key value>
 ikev2 local-authentication pre-shared-key <key value>
```

创建用于 IPSec 连接的 IKEv2 策略配置。IKEv2 策略块设置用于 IKE 交换的参数。在此块中，设置了以下参数：
* 加密算法 - 对于此示例，设置为 AES-256
* 完整性算法 - 对于此示例，设置为 SHA256
* Diffie-Hellman 组 - IPsec 使用 Diffie-Hellman 算法在同级之间生成初始加密密钥。在此示例中，设置为组 14
* 伪随机函数 (PRF) - IKEv2 需要单独的方法用作算法来派生 IKEv2 隧道加密所需的密钥资料和散列操作。这称为伪随机函数，设置为 SHA
* SA 生命周期 - 设置安全性关联的生命周期（在此周期之后将执行重新连接）。设置为 36,000 秒。
* 操作类型 - 将此值保留为缺省值“双向”。（在“show running”显示内容中不会显式出现。）

如以下代码示例中所示，此样本策略使用 AES-256 来加密安全通道。SHA512 散列算法用于验证远程同级的身份，Diffie-Hellman 组 14 用于生成密钥。组 14 使用 2048 位加密块。最后，安全性关联的生命周期设置为 36,000 秒。

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* 定义 VPN 的访问列表和加密映射：

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## 创建与本地 IBM Cloud VPC 的安全连接
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

要创建安全连接，需要在 VPC 中创建 VPN 连接，这类似于 2 个 VPC 的示例。

* 在 VPC 子网上创建 VPN 网关，并创建 VPC 与 Cisco ASAv 之间的 VPN 连接，将 `local_cidrs` 设置为 VPC 上的子网值，将 `peer_cidrs` 设置为 Cisco ASAv 上的子网值。

创建 VPN 网关时，网关状态显示为 `pending`，创建完成后，状态会变为 `available`。创建过程可能需要一些时间。
{:note}


![在此输入图像描述](./images/vpc-vpn-asav-connection.png)

### 检查安全连接的状态
{: #cisco-check-the-status-of-the-secure-connection}

可以通过 IBM Cloud 控制台来检查连接的状态。此外，还可以尝试使用 VSI 执行站点到站点的 `ping` 操作。

![在此输入图像描述](./images/vpc-vpn-asav-status.png)
