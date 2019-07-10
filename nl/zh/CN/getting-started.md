---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-06-06"

keywords: provisioning, resources, permissions

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# 入门教程
{: #getting-started}

开始使用 {{site.data.keyword.cloud}} 虚拟私有云联网：

1. 创建虚拟私有云。
2. 在虚拟私有云的一个或多个专区中创建一个或多个子网。
3. 如果希望子网上的资源有权访问因特网或支持通过因特网访问子网上的资源，请在子网上创建公共网关 (PGW)。

## 先决条件

 * **用户许可权**：确保用户具有足够的许可权来创建和管理 VPC 中的资源。有关必需许可权的列表，请参阅[授予 VPC 用户所需的许可权](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources)。

## 使用 UI、CLI 或 REST API 来供应 VPC 网络资源

可以通过 UI、CLI 或 REST API 来供应和管理 VPC 网络资源。

* 要通过用户界面进行访问，请登录到 [IBM Cloud 控制台 ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")]( https://{DomainName}/vpc){: new_window}。如果需要帮助，请遵循 [UI 指南](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)进行操作。
* 要使用命令行界面，请遵循 [Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 示例。
* 对于更高级的用户，可以直接调用 [REST API](https://{DomainName}/apidocs/vpc-on-classic)。请遵循[示例代码](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)教程开始使用 REST API。

## 后续步骤

供应虚拟专用云联网后，请探索更多信息。

* [创建和管理虚拟服务器实例](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [使用安全组](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [使用网络 ACL](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [使用 VPN](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [使用负载均衡器](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
