---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-06-04"

keywords: VPN, network, encryption, authentication, algorithm, IKE, IPsec, policies, gateway, auto-negotiation

subcollection: vpc-on-classic-network


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# 将 VPN 与 VPC 配合使用
{: #--using-vpn-with-your-vpc}
[comment]: # (链接帮助主题)

通过 {{site.data.keyword.cloud}} VPC VPN 服务，您能以安全方式连接专用网络。您可以使用 VPN 在 VPC 与内部部署专用网络之间或与其他 VPC 之间设置 IPsec 站点到站点隧道。

对于当前 {{site.data.keyword.cloud}} VPC 发行版，仅支持基于策略的路由。

## 功能
{: #vpn-features}

* IKEv1 和 IKEv2
* 认证算法：`md5`、`sha1` 和 `sha256`
* 加密算法：`3des`、`aes128` 和 `aes256`
* Diffie-Hellman (DH) 组：2、5 和 14
* IKE 协商方式：主方式
* IPSec 变换协议：ESP
* IPSec 封装方式：隧道
* 完美前向保密 (PFS)
* 失效同级检测
* 路由：基于策略
* 认证方式：预先共享密钥
* 仅在活动/备用方式下支持 HA

## 可用 API
{: #apis-available}

以下部分提供了有关可用于 IBM Cloud VPC 环境中 VPN 的 API 的详细信息。请参阅 [VPC REST API](https://{DomainName}/apidocs/vpc-on-classic#list-all-ike-policies) 页面，以获取更多详细信息。

### VPN 网关和 VPN 连接
{: #vpn-gateways-and-vpn-connections}

|描述|API|
|----------------------------|-------------|
|创建 VPN 网关|POST /vpn_gateways|
|检索多个 VPN 网关|GET /vpn_gateways|
|检索一个 VPN 网关|GET /vpn_gateways/{id}|
|删除 VPN 网关|DELETE /vpn_gateways/{id}|
|更新 VPN 网关|PATCH /vpn_gateways/{id}|
|创建新的 VPN 连接|POST /vpn_gateways/{vpn_gateway_id}/connections|
|检索多个 VPN 连接|GET /vpn_gateways/{vpn_gateway_id}/connections|
|检索一个 VPN 连接|GET /vpn_gateways/{vpn_gateway_id}/connections/{id}|
|删除 VPN 连接|DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}|
|更新 VPN 连接|PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id}|
|检索 VPN 连接的所有本地 CIDR|GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs|
|从 VPN 连接中删除本地 CIDR|DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length}|
|检查 VPN 连接上是否存在特定本地 CIDR|GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length}|
|在 VPN 连接上设置本地 CIDR|PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length}|
|检索 VPN 连接的所有同级 CIDR|GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs|
|从 VPN 连接中删除同级 CIDR|DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length}|
|检查 VPN 连接上是否存在特定同级 CIDR|GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length}|
|在 VPN 连接上设置同级 CIDR|PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length}|

### IKE 策略
{: #ike-policies}

|描述|API|
|-----------------------------|--------------|
|检索所有 IKE 策略|GET /ike_policies|
|创建 IKE 策略|POST /ike_policies|
|删除 IKE 策略|DELETE /ike_policies/{id}|
|检索 IKE 策略|GET /ike_policies/{id}|
|更新 IKE 策略|PATCH /ike_policies/{id}|
|检索使用指定 IKE 策略的所有连接|GET /ike_policies/{id}/connections|

### IPsec 策略
{: #ipsec-policies}

|描述|API|
|---------------------|-------------|
|检索所有 IPSec 策略|GET /ipsec_policies|
|创建 IPSec 策略|POST /ipsec_policies|
|删除 IPSec 策略|DELETE /ipsec_policies/{id}|
|检索 IPSec 策略|GET /ipsec_policies/{id}|
|更新 IPSec 策略|PATCH /ipsec_policies/{id}|
|检索使用指定 IPsec 策略的所有连接|GET /ipsec_policies/{id}/connections|

## VPN 示例
{: #vpn-example}

在以下示例中，您能够使用 VPN 将两个 VPC 连接在一起，这意味着您可以将两个独立 VPC 中的子网连接在一起，就像它们是单个网络一样。子网的 IP 地址不能重叠。下图说明了此场景（在每个 VPC 中添加了一些 VM）：

![VPN for IBM VPC](images/vpc-vpn.svg "VPN for IBM VPC")

### 示例步骤
{: #vpn-example-steps}

以下示例步骤跳过了使用 IBM Cloud API 或 CLI 来创建 VPC 的先决条件步骤。有关更多信息，请参阅[入门](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)和[使用 API 设置 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

（可选）可以使用 UI 来创建 VPN 网关。相关步骤可以在[控制台教程文档](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)中找到。

#### 步骤 1. 在 VPC 子网中创建 VPN 网关
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

样本输出：
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

确保保存以下字段以供后续步骤使用。
* `id`。这是 VPN 网关标识，将作为 `$gwid1` 引用。
* `address`。这是 VPN 网关的公共 IP 地址，将作为 `$gwaddress1` 引用。

创建 VPN 网关时，网关状态显示为 `pending`，创建完成后，状态会变为 `available`。创建过程可能需要一些时间。
{: note}


您可以使用以下命令来检查网关的状态：

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### 步骤 2. 在不同的 VPC 上创建第二个 VPN 网关
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

样本输出：
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

确保保存以下字段以供后续步骤使用。
* `id`。这是 VPN 网关标识，将作为 `$gwid2` 引用。
* `address`。这是 VPN 网关的公共 IP 地址，将作为 `$gwaddress2` 引用。


#### 步骤 3. 创建从第一个 VPN 网关到第二个 VPN 网关的 VPN 连接
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

创建连接时，将 `local_ciders` 设置为 **VPC one** 上的子网，并将 `peer_cidrs` 设置为 **VPC two** 上的子网。

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

样本输出：
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
        "interval": 30,
        "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 步骤 4. 创建从第二个 VPN 网关到第一个 VPN 网关的 VPN 连接
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

创建连接时，将 `local_ciders` 设置为 **VPC two** 上的子网，并将 `peer_cidrs` 设置为 **VPC one** 上的子网。

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress1,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

样本输出：
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
        "interval": 30,
        "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 步骤 5. 验证连接
{: #step-5-verify-connectivity}

建立 VPN 连接后，您将能够从子网 1 访问子网 2 上的实例，也可以从子网 2 访问子网 1 上的实例。

可以检查 VPN 连接的状态，如下所示：
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

样本输出：
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## 配额
{: #see-vpn-quotas}

请参阅 [VPC 配额](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas)主题以获取 VPN 配额的信息。

## 策略自动协商
{: #policy-auto-negotiation}

使用 IKE 和 IPsec 策略来配置 VPN 连接是可选的。如果未选择任何策略，那么将自动为过程选择缺省建议，这称为_自动协商_。 

IBM Cloud 自动协商使用 **IKEv2**，因此内部部署设备还必须使用 **IKEv2**。如果内部部署设备不支持 **IKEv2**，请使用定制的 IKE 策略。
{: note}

### IKE 自动协商（第 1 阶段）
{: #ike-auto-negotiation-phase-1}

以下加密、认证和 Diffie-Hellman 组选项能以任意组合使用：

|    |加密|认证|DH 组|
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### IPsec 自动协商（第 2 阶段）
{: #ipsec-auto-negotiation-phase-2}

以下加密和认证选项能以任意组合使用：

|    |加密|认证|DH 组|
|----|------------|----------------|----------|
| 1  | aes128 | sha1   |已禁用|
| 2  | aes256 | sha256 |已禁用|
| 3  | 3des   | md5    |已禁用|

## 常见问题
{: #vpn-faq}

**创建 VPN 网关时，可以同时创建 VPN 连接吗？**

如果使用的是 API 或 CLI，那么必须先创建 VPN 网关，再创建 VPN 连接。在 IBM Cloud 控制台中，可以同时创建网关和连接。

**如果删除连接了 VPN 连接的 VPN 网关，这些连接会发生什么情况？**

VPN 连接会与 VPN 网关一起删除。

**删除 VPN 网关或 VPN 连接时，会删除 IKE 或 IPSec 策略吗？**

不会，IKE 和 IPSec 策略可以应用于多个连接。

**如果尝试删除网关所在的子网，VPN 网关会发生什么情况？**

如果子网存在任何实例（包括 VPN 网关），那么无法删除该子网。

**有缺省 IKE 和 IPsec 策略吗？**

在不引用策略标识（IKE 或 IPsec）的情况下创建 VPN 连接时，将使用自动协商。

**为什么在 VPN 网关供应期间需要选择一个子网？**

子网将 VPN 网关连接到 VPC 中的其他资源。建议的最佳做法是为 VPN 网关创建专用子网，在此子网上没有其他 VPC 实例，以确保子网中有足够的可用专用 IP。VPN 网关需要 8 个专用 IP 地址来容纳 HA 和卷动升级。

**VPN 网关位于哪个专区？**

VPN 网关位于供应期间选择的子网所在的专区中。请记住，VPN 网关只为同一专区和同一 VPC 中的 VPC 实例提供服务。因此，其他专区中的 VPC 实例无法利用该 VPN 网关与内部部署专用网络进行通信。要实现专区容错，必须为每个专区部署一个 VPN 网关。

**如果要在用于部署 VPN 网关的子网上使用 ACL，应该怎么做？**

请确保以下 ACL 规则已就位，以允许管理流量和 VPN 通道流量：

* **入站规则**
    - 允许协议 TCP，源端口 9091
    - 允许协议 TCP，源端口 10514
    - 允许协议 TCP，源端口 443
    - 允许协议 TCP，源端口 80
    - 允许协议 TCP，源端口 53
    - 允许协议 UDP，源端口 53
    - 允许协议 ALL，源 IP 为 VPN 同级网关公共 IP
    - 允许协议 TCP，目标端口 443
    - 允许协议 TCP，目标端口 56500
    - 允许 VPC 中的实例与内部部署专用网络之间的流量
    - 允许 ICMP 流量

* **出站规则**
   - 允许所有流量

**如果要在需要与内部部署专用网络进行通信的子网上使用 ACL 或安全组，应该怎么做？**

您需要确保存在 ACL 规则或安全组规则，以允许 VPC 中的实例与内部部署专用网络之间的流量。

**用于 VPC 的 VPN 支持 HA 配置吗？**

是的，支持活动/备用配置中的 HA。

**有计划支持 SSL VPN 吗？**

没有，仅支持 IPsec 站点到站点。

**站点到站点 VPNaaS 的吞吐量有任何上限吗？**

我们支持最高 650 Mbps 的吞吐量。

**VPNaaS 支持 PSK 和基于证书的 IKE 认证吗？**

仅支持 PSK 认证。

**您是否可以将 VPN for VPC 用作 IBM Cloud Infrastructure Classic 的 VPN 网关？**

不能，要在 IBM Cloud Infrastructure Classic 环境中使用 VPN 网关，必须使用 [IPsec VPN](https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn)。
