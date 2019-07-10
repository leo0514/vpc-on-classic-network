---

copyright:
  years: 2019

lastupdated: "2019-05-20"

keywords: security groups, traffic, firewall, stateful, filtering

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
{:DomainName: data-hd-keyref="DomainName"}

# 使用安全组
{: #using-security-groups}
[comment]: # (链接帮助主题)

通过安全组，您可以方便地应用规则来根据 IP 地址对虚拟服务器实例 (VSI) 的每个网络接口执行过滤。创建新的安全组资源时，应对其进行更新，以创建所需的网络流量模式。

有关安全组与 ACL 特征的比较，请参阅[比较表](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists)。

缺省情况下，安全组会拒绝所有流量。向安全组添加规则时，会定义该安全组允许的流量。

规则是_有状态的_，这意味着会自动允许用于响应所允许流量的逆向流量。例如，允许端口 80 上入站 TCP 流量的规则还会允许将端口 80 上的回复出站 TCP 流量返回给发起主机，而无需额外的规则。

安全组的作用域限定为单个 VPC。此作用域限定意味着一个安全组_只能_连接到同一 VPC 中 VSI 的网络接口。

创建 VSI 时，如果未指定任何安全组，那么该 VSI 的主网络接口将放入该 VSI 的 VPC 的_缺省_安全组中。此 {{site.data.keyword.cloud}} VPC 发行版定义了允许特定流量的缺省安全组。有关更多信息，请参阅[更新缺省安全组](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-updating-the-default-security-group)。

可以使用 REST API、CLI 或 UI 来设置安全组：

* [使用 API 设置安全组](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-apis)
* [使用 CLI 设置安全组](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [使用 UI 设置安全组](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
