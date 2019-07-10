---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: capabilities, use cases, subnets, VPN, connections, reserved, IP, IPv4, floating

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

# 關於 VPC 的網路
{: #about-networking-for-vpc}

在本文件中，您會看到一些概念，其說明在 {{site.data.keyword.cloud}} VPC 內關於子網路、浮動 IP 位址及 VPN 連線的使用案例和功能。

![IBM VPC 連線功能及安全](images/vpc-connectivity-and-security.svg "IBM VPC 連線功能及安全"){: caption="圖：您可以用子網路來細分 Virtual Private Cloud，如果想要的話，每個子網路可以連接公用網際網路。" caption-side="top"}

如下圖所示：

* 子網路可以透過選用的「公用閘道 (PGW)」連接至公用網際網路。
* 您可以將「浮動 IP 位址 (FIP)」指派給任何 VSI，以從網際網路連接它，反之亦然。
* IBM Cloud VPC 中的子網路提供專用連線功能；它們可以透過專用鏈結彼此交談。您不需要設定任何路徑。
* 如需相關資訊，請參閱[關於 VPC 基礎架構](/docs/vpc-on-classic?topic=vpc-on-classic-about)。

## 術語
{: #terminology}

[名詞解釋](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary)包含本文件針對 IBM Cloud VPC 所使用術語的定義及相關資訊。

## VPC 中的子網路性質
{: #characteristics-of-subnets-in-the vpc}

子網路是由指定的 IP 位址範圍（CIDR 區塊）組成。子網路會連結至單一區域，而且它們不能跨越多個區域或地區。不過，子網路可以跨越其 Virtual Private Cloud 內的整個區域抽象項目。同一個 IBM Cloud VPC 中的子網路可以彼此連接。

### 區域
{: #zones}

「區域」是一種抽象概念，用來協助改良容錯及減少延遲。區域可保證下列內容：

 * 每個區域都是一個獨立的錯誤網域，且一個地區中的兩個區域極不可能同時失敗。
 * 一個地區中的兩個區域之間的資料流量，其延遲 < 2 毫秒。

### 保留的 IP 位址
{: #reserved-ip-addresses}

在操作 Virtual Private Cloud 時，某些 IP 位址已保留供 IBM 使用。以下是保留的位址（給定的 IP 位址假設子網路的 CIDR 範圍是 10.10.10.0/24）：

  * CIDR 範圍中的第一個位址 (10.10.10.0)：網址
  * CIDR 範圍中的第二個位址 (10.10.10.1)：閘道位址
  * CIDR 範圍中的第三個位址 (10.10.10.2)：由 IBM 保留
  * CIDR 範圍中的第四個位址 (10.10.10.3)：由 IBM 保留供未來使用
  * CIDR 範圍中的最後一個位址 (10.10.10.255)：網路播送位址

### 對子網路的外部連線功能使用「公用閘道」
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

**公用閘道 (PGW)** 可讓子網路（所有 VSI 都連接至子網路）連接至網際網路。請注意，依預設，子網路是專用的；不過，您可以選擇性地建立 PGW，並將子網路連接至 PGW。子網路連接至 PGW 之後，該子網路中的所有 VSI 就可以連接至網際網路。

PGW 使用_多對一 NAT_，這表示含有專用位址的數千個 VSI 將使用 1 個公用 IP 位址來與公用網際網路進行交談。PGW 不會讓網際網路起始與那些實例的連線。請使用 API 將子網路與 PGW 連接及分離。

下圖彙總閘道服務的範圍。

![閘道服務](images/scope-of-gateway-services.png)

### 對 VSI 的外部連線功能使用浮動 IP 位址
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

**浮動 IP 位址**是系統所提供的 IP 位址，可從公用網際網路連接它們。

您可以從 IBM 所提供的可用「浮動 IP 位址」儲存區中保留一個「浮動 IP 位址」，您可以將它與相同 Virtual Private Cloud 中的任何實例建立關聯或取消關聯，方法是透過該實例的 vNIC 來進行。任何「浮動 IP 位址」都可以與各種類型的虛擬伺服器實例 (VSI) 相關聯，例如，負載平衡器或 VPN 閘道。

您的「浮動 IP 位址」無法與多個介面相關聯。您必須在 VSI 上指定要與該個別「浮動 IP」相關聯的介面。該介面也會有專用 IP 位址。後端系統在「浮動 IP」和該介面的專用 IP 之間執行_一對一 NAT_ 作業。只有在您特別要求釋放動作時，才會釋放「浮動 IP」。

**附註：**
* **建立「浮動 IP 位址」與 VSI 的關聯，可從前面提及的 PGW「多對一 NAT」中移除 VSI。**
* **目前，「浮動 IP」僅支援 IPv4 位址。**
* **您不能使用自己的公用 IP 位址作為「浮動 IP」。**

如需 NAT 作業的相關資訊，請參閱[相關的網際網路 RFC 文件![外部鏈結圖示](../../icons/launch-glyph.svg "外部鏈結圖示")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}。

### 使用 VPN 建立安全的外部連線
{: #use-a-vpn-for-secure-external-connectivity}

Virtual Private Network (VPN) 服務可讓使用者從網際網路安全地連接至其 IBM Cloud VPC。如需逐步指示，請參閱 [IBM 主控台使用者介面手冊](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)。

**VPN 功能**
  * 對於 VPN 服務（點對點 IPSEC VPN）執行 CRUD（建立、讀取、更新及刪除）來處理資料傳送的能力。
  * 將 VPN 服務與客戶之 IBM Cloud VPC 或虛擬網路相關聯的能力。
  * 依靜態定義或動態遞送來識別客戶的現場子網路的能力。
  * 支援安全密碼，例如 SHA256、AES、3DES、IKEv2。
  * 內建服務可靠性。
  * 監視 VPN 連線性能。
  * 同時支援起始器及回應者模式；亦即，可以從通道任一端起始資料流量。
