---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-07"

keywords: vpc, address prefix, region, subnet, zone, reserved, IP, ranges, deleting, creating, CIDR

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# 瞭解 IP 位址範圍、位址字首、地區及子網路
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (鏈結的說明主題)

本文件討論 {{site.data.keyword.cloud}} VPC 之地區、位址字首與子網路間的關係：

* VPC 會部署並連結至地區。
* 在這一個地區內，VPC 可以跨越多個區域。
* 位址字首可讓您在 VPC 跨越的區域之間進行通訊。每一個 VPC 都會為它所跨越的每個區域，取得一個簡短的預設位址字首。
* 子網路是在位址字首的範圍內建立的，這表示必須將子網路完全包含在單一的現有位址字首內。
* 如果您針對子網路使用 [RFC 1918](https://tools.ietf.org/html/rfc1918) 所定義範圍之外的 IP 範圍（`10.0.0.0/8`、`172.16.0.0/12` 或 `192.168.0.0/16`），則連接至該子網路的實例可能無法連接公用網際網路的某些部分。

## IBM Cloud VPC 和地區
{: #ibm-cloud-vpc-and-regions}

{{site.data.keyword.cloud}} VPC 部署在某個地區中，但它可能會跨越多個區域。每個地區包含多個區域，分別代表獨立錯誤網域。

## IBM Cloud VPC 及位址字首
{: #ibm-cloud-vpc-and-address-prefixes}

位址字首可讓您在不同區域的 {{site.data.keyword.cloud_notm}} VPC 實例之間進行通訊。它們提供_隱含路由器_ 所需的遞送資訊，以將資料傳送至目的地實例所在的區域。每個子網路都必須包含在某個位址字首內。{{site.data.keyword.cloud_notm}} VPC 不支援「本端」或「無法連接」子網路的概念。

_隱含路由器_ 是 VPC 內建立的所有子網路之間的繼承網路連線。
{: note}

每一個 {{site.data.keyword.cloud_notm}} VPC 在每個區域最多能夠有五個位址字首。為了協助您開始使用，{{site.data.keyword.cloud_notm}} VPC 會定義每個區域的預設位址字首（請參閱下列表格），不過，最佳作法是您應該在部署 VPC 之前，先設計 VPC 定址方案。

### VPC 預設位址字首
{: #default-vpc-address-prefixes}

建立新的 VPC 時，會根據地區及區域指派預設位址字首，如下所示。

[標準存取 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) 具有一組不同的預設位址字首。
{: important}

 區域 | 位址字首  
---------------|---------------
`us-south-1`   | `10.240.0.0/18`
`us-south-2`   | `10.240.64.0/18`
`us-south-3`   | `10.240.128.0/18`
`eu-de-1`      | `10.243.0.0/18`
`eu-de-2`      | `10.243.64.0/18`
`eu-de-3`      | `10.243.128.0/18`
`jp-tok-1`     | `10.244.0.0/18`
`jp-tok-2`     | `10.244.64.0/18`
`jp-tok-3`     | `10.244.128.0/18`

將指派不同的預設字首給新區域或地區。

### 位址字首和 IBM Cloud 主控台使用者介面
{: #address-prefixes-and-the-ibm-cloud-console-ui}

當您使用 IBM Cloud 主控台使用者介面來建立 VPC 時，系統會自動選取您的位址字首，且您需要在該預設字首內建立子網路。如果這個位址架構不符合您的需求，您可以在建立 VPC 之後自訂位址字首。然後，您可以在自訂的位址字首中建立子網路，並刪除您使用預設字首所建立的子網路。

需要此暫行解決方法，才能透過「IBM Cloud 主控台」使用者介面使用 BYOIP。
{:note}

## IBM Cloud VPC 及子網路
{: #ibm-cloud-vpc-and-subnets}

您可以將 {{site.data.keyword.cloud_notm}} VPC 分配為多個子網路。{{site.data.keyword.cloud_notm}} VPC 中的所有子網路都可以透過隱含路由器的專用 L3 遞送來連接另一個子網路。您不需要設定任何路由器或路徑。

關於 VPC 中子網路的有用資訊：

* 子網路是由您指定的 IP 位址範圍組成。
* 子網路會連結至單一區域，它不能跨越多個區域或地區。
* 子網路可以跨越 {{site.data.keyword.cloud_notm}} VPC 中的整個區域。
* 您必須先建立 VPC，然後才能在該 VPC 內建立子網路。
* 不提供 IPv6 支援。
* 您可以將 VSI 與子網路相關聯或取消關聯。（它需要您新增 vNIC 並選取頻寬。）
* 每一個子網路都必須包含在子網路連結所在區域所屬的位址字首內。

您可以將自己的公用 IPv4 位址範圍 (BYOIP) 帶至 {{site.data.keyword.cloud_notm}} VPC 帳戶。當您使用 BYOIP 時，{{site.data.keyword.cloud_notm}} 必須在 {{site.data.keyword.cloud_notm}} 資源上配置這些 IPv4 位址，其將在所提供的位址中來回傳送封包。因此，當您在 {{site.data.keyword.cloud_notm}} 上使用所提供的 IPv4 範圍時，這些 IP 位址有可能在您使用此服務時向 IBM 支援人員及協力廠商公開。
{:important}

![IBM Cloud VPC 概觀](images/vpc-experience.svg "IBM Cloud VPC 概觀"){: caption="圖：VPC 的圖形表示，顯示區域、地區及子網路。" caption-side="top"}

### 使用子網路的位址字首
{: #using-address-prefixes-for-subnets}

每個子網路都必須存在於位址字首內。
 * 對於新的子網路，您可以從現有位址字首中挑選您的 IP 位址範圍。
 * 如果區域的位址字首不適合，可以編輯其中一個現有位址字首，或為區域新增一個新的位址字首（使用 API、CLI 或使用者介面）。

### 可用的 IP 位址
{: #available-ip-addresses}

這些是可用的 IP 位址，如 **RFC 1918** 中所定義：

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

如果您使用的 IP 範圍在子網路的可容許範圍之外（在之前的章節中提供），則連接至該子網路的實例可能無法連接到公用網際網路的某些部分。

### 其他有關建立子網路的資訊
{: #more-about-creating-a-subnet}

您可以使用兩種方式來指定 VPC 的子網路：
  * 您可以提供所需的子網路大小來建立子網路，例如支援的位址數目（例如，1024）。
  * 您可以提供 CIDR 範圍（例如 10.0.0.8/29）來建立子網路。

以如何使用 CIDR 指定 1024 區塊為例，如果您指定 CIDR 範圍而非子網路大小，則 IPv4 區塊 `192.168.100.0/22` 代表從 `192.168.100.0`到 `192.168.103.255` 的 1024 個 IPv4 位址。
{:tip}

