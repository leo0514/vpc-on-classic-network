---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: VRF, router, hypervisor, address prefixes, classic access, implicit router, packet flows, NAT, data flows

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

# VPC 幕後
{: #vpc-behind-the-curtain}

此頁面對於 VPC 網路幕後發生的真相提供更詳細的概念圖。讀者應具備一些網路背景。

## 網路隔離
{: #network-isolation}

VPC 網路隔離在下列三個層次上進行：

* **Hypervisor**：VSI（虛擬伺服器實例）是由 Hypervisor 本身予以隔離。如果 VSI 不在相同的 VPC 中，則 VSI 無法直接連接相同 Hypervisor 管理的其他 VSI。

* **網路**：此隔離是透過使用**虛擬網路 ID** (VNI) 在網路層次上進行。這些 ID 的範圍限於本端區域。這些 VNI 會新增至所有進入任何 VPC 區域的資料封包：從 Hypervisor 進入（由 VSI 傳送時），或從雲端進入區域（由隱含的遞送功能傳送時）。

離開區域的封包會刪除 VNI。當封包到達其目的地區域，並透過隱含的遞送功能輸入時，隱含的路由器一律會為該區域新增適當的 VNI。
{: note}

* **路由器**：_隱含的路由器功能_ 藉由在雲端骨幹中提供**虛擬遞送功能** (VRF) 以及含 MPLS（多重通訊協定標籤切換）的 VPN，來提供每個 VPC 的隔離。每個 VPC 的 VRF 都有唯一 ID，此隔離可讓每個 VPC 存取它自己的 IPv4 位址空間副本。MPLS VPN 可聯合雲端的所有邊緣：「標準基礎架構」、「直接鏈結」及 VPC。

## 位址字首
{: #address-prefixes}

位址字首是 VPC 隱含遞送功能用來尋找_目的地 VSI_ 的摘要資訊（不論目的地 VSI 所在的可用性區域為何）。位址字首的主要功能是將透過 MPLS VPN 的遞送最佳化，而避免病態遞送的狀況。VPC 中建立的所有子網路都必須內含在位址字首中，以從 VPC 中的所有其他 VSI 連接 VPC 中的所有 VSI。

## 資料封包流程和隱含路由器
{: #data-packet-flows-and-the-implicit-router}

VPC 中出現六種不同類型的 VSI 資料封包流程。這些流程依複雜性遞增顯示如下：

* 內部網路、內部主機（相同的 Hypervisor）
* 內部子網路、主機之間
* 子網路之間、內部區域
* 子網路之間、區域之間
* 額外 VPC 服務（適用於 IaaS 或 CSE 存取）
* 額外 VPC 網際網路（適用於網際網路存取）

**內部子網路、內部主機**資料流程：這些是最簡單的。在 Hypervisor 上的 VSI 之間的封包流程，沒有封包會離開 Hypervisor。

**內部子網路、主機之間**資料流程：這些流程涉及離開 Hypervisor 的封包。每個封包會以適當的 VNI（虛擬化網路 ID）標記，以確保資料隔離，然後傳送給管理目的地 VSI 的目的地 Hypervisor。目的地 Hypervisor 會除去 VNI，並將資料封包轉遞至目的地 VSI。

**子網路之間、內部區域**資料流程：這些流程涉及利用 VPC 隱含路由器功能的封包，該功能會連接 VPC 中建立的所有子網路。它會將資料封包遞送至正確的目的地 Hypervisor。如果目的地 Hypervisor 與來源 Hypervisor 不同，則會以適當的 VNI 標記資料封包，並將它傳送至目的地 Hypervisor。該處的 VNI 已被刪除，而資料封包則轉遞至目的地 VSI。（這些最後步驟與前一類型的資料流程所說明的步驟相同。）

**子網路之間、區域之間**資料流程：對於這些資料流程，隱含路由器功能會移除 VNI 並轉遞 VPC 之 MPLS VPN 中的封包，以透過雲端骨幹進行傳輸。在目的地區域中，隱含路由器功能會以適當的 VNI 標記資料封包。然後，封包會轉遞至目的地 Hypervisor，該處的 VNI（如先前所述）已遭刪除，這樣資料封包才可以轉遞至目的地 VSI。

**額外 VPC 服務**資料流程：預定用於 IaaS 或「IBM Cloud 服務端點 (CSE)」服務的封包將利用 VPC 的隱含路由器功能，它們也可以利用網址轉換 (NAT) 功能。轉換功能會將 VSI 位址取代為 IPv4 位址，以向所要求的 IaaS 或 CSE 服務識別 VPC。

**額外 vpc 網際網路**資料流程：預定用於網際網路的封包是最複雜的。除了利用 VPC 的隱含路由器功能之外，每個流程都還會仰賴其中一個隱含路由器的兩個網址轉換 (NAT) 功能：

  * 透過公用閘道功能來服務其連接的所有子網路的明確一對多 NAT。
  * 指派給個別 VSI 的一對一 NAT。

經過 NAT 轉換之後，隱含的路由器會利用雲端骨幹，將這些預定用於網際網路的封包轉遞至網際網路。

## 標準存取
{: #classic-access}

VPC 的[**標準存取**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc)特性是透過重複使用 {{site.data.keyword.cloud}} 標準基礎架構帳戶中的 VRF ID 作為 VPC 的 VRF ID 來達成。此實作容許 VPC 的隱含路由器功能結合「標準基礎架構帳戶」使用的相同 MPLS VPN。因此，VPC 可以存取標準資源及任何可透過現有「直接鏈結」連線連接的其他資源。
