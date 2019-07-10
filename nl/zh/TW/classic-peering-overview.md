---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: peering, classic, infrastructure, VRF, resources

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

# VPC 的標準對等作業
{: #setting-up-classic-peering-for-vpc}

{{site.data.keyword.cloud}} VPC 有能力與標準 IBM Cloud 資源進行對等作業。您的帳戶必須符合幾項需求，才能進行對等作業。

若要取得關於建立已啟用標準對等作業的 VPC 的循序漸進步驟，請參閱 [VPC 基礎架構文件](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc)，其中涵蓋了這些程序的其他詳細資料。

## 需求
{: #classic-peering-requirements}

1. 您的 VPC 必須在建立時就指定為「標準對等作業」，因為之後的任何時間都不能再進行指定。

2. 您所要與 VPC 進行對等作業的已鏈結帳戶必須已啟用 VRF。如需 VRF 的相關資訊，請參閱[這個解釋文件](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud)。

3. 您也必須從您在「標準基礎架構」端上使用的每一個實例，將路徑新增回已啟用標準的 VPC。

## 限制
{: #classic-peering-restrictions}

對於標準對等作業，每個區域只能啟用一個 VPC。
