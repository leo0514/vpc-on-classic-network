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

# 入門指導教學
{: #getting-started}

若要開始使用 {{site.data.keyword.cloud}} Virtual Private Cloud 的網路功能，請執行下列動作：

1. 建立 Virtual Private Cloud。
2. 在一個以上區域的 Virtual Private Cloud 中建立一個以上的子網路。
3. 如果您希望子網路上的資源可以存取網際網路，請在子網路上建立公用閘道 (PGW)，反之亦然。

## 必要條件

 * **使用者許可權**：確保您的使用者具有足夠的許可權可在 VPC 中建立及管理資源。如需必要許可權的清單，請參閱[授與 VPC 使用者所需的許可權](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources)。

## 使用使用者介面、CLI 或 REST API 來佈建 VPC 網路資源

透過使用者介面、CLI 或 REST API 可以執行 VPC 網路資源的佈建及管理。

* 若要透過使用者介面存取，請登入 [IBM Cloud 主控台![外部鏈結圖示](../../icons/launch-glyph.svg "外部鏈結圖示")]( https://{DomainName}/vpc){: new_window}。如需協助，請遵循[使用者介面手冊](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)。
* 若要使用指令行介面，請遵循 [Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 範例。
* 若為更高階的使用者，您可以直接呼叫 [REST API](https://{DomainName}/apidocs/vpc-on-classic)。請遵循[程式碼範例](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)指導教學，開始使用 REST API。

## 下一步

在佈建 Virtual Private Cloud 網路之後，可進一步探索。

* [建立及管理虛擬伺服器實例](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [使用安全群組](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [使用網路 ACL](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [使用 VPN](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [使用 Load Balancers](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
