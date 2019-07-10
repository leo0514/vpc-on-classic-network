---

copyright:
  years: 2019

lastupdated: "2019-06-11"

keywords: ACLs, network, CLI, example, tutorial, firewall, subnet, inbound, outbound, rule

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


# 设置网络 ACL
{: #setting-up-network-acls}
[comment]: # (链接帮助主题)

通过使用 {{site.data.keyword.cloud}} 虚拟私有云中提供的访问控制表 (ACL) 功能，可以控制与云上关键业务工作负载相关的所有入局和出局流量。ACL 是与安全组类似的内置虚拟防火墙。但与安全组不同的是，ACL 规则控制的是与_子网_之间的进出流量，而不是与_实例_之间的进出流量。

有关安全组与 ACL 特征的比较，请参阅[比较表](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists)。

本文档中给出的示例说明了如何使用 CLI 在 VPC 中创建网络 ACL 以保护子网。

## API 和 CLI 可用于处理 ACL
{: #apis-and-clis-are-available-for-acls}

您可以通过 API、CLI 或 UI 来设置和管理 ACL。

* 请参阅 [API 参考](https://{DomainName}/apidocs/vpc-on-classic)，以获取每个 API 的参数、请求主体和响应的详细信息。

* 请参阅此 [CLI 示例](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)，以获取命令详细信息、安装 CLI 插件的步骤以及使用 VPC CLI 的先决条件步骤。

* 请参阅 [使用 UI 配置 ACL](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-acl)，以获取有关如何在 {{site.data.keyword.cloud_notm}} 控制台中设置 ACL 的信息。

ACL 规则控制进出_子网_的流量。看起来，VPC API、CLI 和 UI 支持为入站规则定制**目标 IP 范围**并为出站规则定制**源 IP 范围**，但是这些地址是不可定制的。**入站 ACL 规则的目标 IP 范围和出站 ACL 规则的源 IP 范围都各自绑定到连接的子网**。因此，实际上，**0.0.0.0/0** 应当用作入站规则的目标 IP 和出站规则的源 IP。
{: note}

## 使用 ACL 和 ACL 规则
{: #working-with-acls-and-acl-rules}

要使 ACL 生效，您应创建规则来确定如何处理入站和出站网络流量。您可以创建多个入站和出站规则。请参阅[配额](/docs/vpc-on-classic?topic=vpc-on-classic-quotas)，以获取有关可以创建的规则数的具体信息。

* 通过入站规则，可以使用指定的协议和端口来允许或拒绝从源 IP 范围流出的流量。目标 IP 范围由连接的子网确定。
* 通过出站规则，可以使用指定的协议和端口来允许或拒绝进入目标 IP 范围的流量。源 IP 范围由连接的子网确定。
* ACL 规则按顺序进行考虑。仅当具有较低优先级号（如 1）的规则不匹配时，才会对具有更高优先级的规则（如 2）求值。
* 入站规则独立于出站规则。
* 当前支持的协议为 TCP、UDP 和 ICMP。还可以使用**所有**选项来指定_所有_或_其他_协议（如果指定了具有更高优先级的规则）。

有关在 ACL 规则中使用 ICMP、TCP 和 UDP 协议的相关信息，请参阅[了解因特网通信协议](/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols)。

### 将 ACL 连接到子网
{: #attaching-an-acl-to-a-subnet}

将 ACL 连接到子网时，有两个选项：

* 可以创建新的子网，并指定要连接的 ACL。如果未指定 ACL，那么将连接缺省网络 ACL。缺省 ACL 允许流至此子网的所有入站流量以及从此子网流出的所有出站流量。
* 可以将 ACL 连接到现有子网。如果其他 ACL 已连接到此子网，那么会先从该 ACL 拆离，然后再连接新的 ACL。

## ACL 演示示例
{: #acl-demo-example}

在以下示例中，您将能够使用命令行界面 (CLI) 创建两个 ACL，并将其与两个子网相关联。下图说明了此场景：

![示例 ACL 场景](images/vpc-acls.png)

如图所示，您有两个 Web 服务器用于处理来自因特网的请求，还有两个您希望向公众隐藏的后端服务器。在此示例中，您将服务器分别放入两个单独的子网 10.10.10.0/24 和 10.10.20.0/24 中，并且您需要允许 Web 服务器与后端服务器交换数据。此外，您还希望允许从后端服务器流出的有限出站流量。

### 示例规则
{: #acl-example-rules}

以下示例规则说明了如何如前所述为基本场景设置 ACL 规则。

作为最佳实践，建议您为细颗粒度规则提供高于粗颗粒度规则的优先级。例如，如果您有一个规则用于阻止来自子网 10.10.30.0/24 的所有流量，并且与更高优先级相匹配，那么将永远不会应用用于允许来自 10.10.30.5 的流量的较低优先级细颗粒度规则。
{:note}

**ACL-1 示例规则**：

|入站/出站|允许/拒绝|源 IP|目标 IP|协议|端口|描述|
|--------------|-----------|------|------|------|------------------|-------|
|入站|允许|0.0.0.0/0|0.0.0.0/0|TCP|80|允许来自因特网的 HTTP 流量|
|入站|允许|0.0.0.0/0|0.0.0.0/0|TCP|443|允许来自因特网的 HTTPS 流量|
|入站|允许|10.10.20.0/24|0.0.0.0/0|所有|所有|允许来自后端服务器所在子网 10.10.20.0/24 的所有入站流量|
|入站|拒绝|0.0.0.0/0|0.0.0.0/0|所有|所有|拒绝其他所有入站流量|
|出站|允许|0.0.0.0/0|0.0.0.0/0|TCP|80|允许流至因特网的 HTTP 流量|
|出站|允许|0.0.0.0/0|0.0.0.0/0|TCP|443|允许流至因特网的 HTTPS 流量|
|出站|允许|0.0.0.0/0|10.10.20.0/24|所有|所有|允许流至后端服务器所在子网 10.10.20.0/24 的所有出站流量|
|出站|拒绝|0.0.0.0/0|0.0.0.0/0|所有|所有|拒绝其他所有出站流量|


**ACL-2 示例规则**：

|入站/出站|允许/拒绝|源 IP|目标 IP|协议|端口|描述|
|--------------|-----------|------|------|------|------------------|--------|
|入站|允许|10.10.10.0/24|0.0.0.0/0|所有|所有|允许来自 Web 服务器所在子网 10.10.10.0/24 的所有入站流量|
|入站|拒绝|0.0.0.0/0|0.0.0.0/0|所有|所有|拒绝其他所有入站流量|
|出站|允许|0.0.0.0/0|0.0.0.0/0|TCP|80|允许流至因特网的 HTTP 流量|
|出站|允许|0.0.0.0/0|0.0.0.0/0|TCP|443|允许流至因特网的 HTTPS 流量|
|出站|允许|0.0.0.0/0|10.10.10.0/24|所有|所有|允许流至 Web 服务器所在子网 10.10.10.0/24 的所有出站流量|
|出站|拒绝|0.0.0.0/0|0.0.0.0/0|所有|所有|拒绝其他所有出站流量|

此示例仅说明的是一般情况。在您的场景中，您还可能希望对流量具有更详细的控制：

* 您可能有网络管理员出于操作目的，需要从远程网络访问 10.10.10.0/24 子网。在这种情况下，您需要允许从因特网流至此子网的 SSH 流量。
* 您可能希望缩小两个子网之间允许的协议作用域。

### 示例步骤
{: #acl-example-steps}

以下示例步骤跳过了使用 CLI 创建 VPC 的先决条件步骤，后者是必须先执行的步骤。有关更多信息，请参阅[使用 CLI 创建 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)。


#### 步骤 1. 创建 ACL
{: #step-1-create-the-acls}

以下 CLI 命令可用于创建名为 `my_web_subnet_acl` 和 `my_backend_subnet_acl` 的两个 ACL：

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

响应包含新创建的 ACL 标识。请保存这两个 ACL 的标识，以便在稍后的命令中使用。可以使用名为 `webacl` 和 `bkacl` 的变量，如下所示：

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### 步骤 2. 检索缺省 ACL 规则
{: #step-2-retrieve-the-default-acl-rules}

添加新规则之前，请检索缺省入站和出站 ACL 规则，以便可以在缺省规则之前插入新规则。

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

响应显示允许所有协议中所有 IPv4 流量的缺省入站和出站规则。

```
Getting rules of network acl ba9e785a-3e10-418a-811c-56cfe5669676 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

将这两个 ACL 规则的标识保存为变量，您将在稍后的命令中使用这些标识。例如，可以使用名为 `inrule` 和 `outrule` 的变量：

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### 步骤 3. 如前所述添加新的 ACL 规则
{: #step-3-add-new-acl-rules-as-decribed}

此部分说明了首先添加入站规则，然后添加出站规则。

在缺省入站规则之前插入新的入站规则。

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

在每个步骤中，将 ACL 规则的标识保存在变量中，在稍后的命令中会使用该标识。例如，可以使用变量名称 `acl200`：

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

现在，将规则添加到 `acl200`：

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

添加更多规则，直到 ACL 设置完成，并将每个标识都保存为一个变量。

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

在缺省出站规则之前插入新的出站规则。

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e $webacl deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $outrule
acl200e="11110627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule100e $webacl allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule $acl200e
acl100e="22220627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20e $webacl allow outbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100e
acl20e="33330627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10e $webacl allow outbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20e
```
{: codeblock}

#### 步骤 4. 使用新创建的 ACL 创建两个子网
{: #step-4-create-the-two-subnets-with-the-newly-created-acl}

您将创建两个子网，以便每个 ACL 与其中一个新子网相关联。

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## 命令列表备忘单
{: #acl-cli-command-list-cheat-sheet}

显示 ACL 的可用 VPC CLI 命令的完整列表：

```
ibmcloud is network-acls
```
{: pre}

查看 ACL 及其元数据（包括规则）：

```
ibmcloud is network-acl $webacl
```
{: pre}

获取缺省入站 ACL 规则：

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## 示例入站 `ping` 规则
{: #acl-example-inbound-ping-rule}

要添加 ACL 规则，下面是在缺省入站规则之前添加 `ping` 入站规则的示例命令：

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
