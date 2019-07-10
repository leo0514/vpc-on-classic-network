---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-29"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# IBM Cloud VPC 中的安全
{: #security-in-your-ibm-cloud-vpc}

您可以使用安全群組 (SG)、使用網路存取控制清單 (ACL) 或同時使用這兩種控制類型來控制網路資料流量，以保持 VPC 與工作負載的安全。請記住：

* 安全群組會控制每個實例 (VSI) 的資料流量。
* 存取控制清單會控制每個子網路的資料流量。

## 安全概觀
{: #security-overview}

* 進出子網路的資料流量可由「存取控制清單 (ACL)」來控制
* 安全群組 (SG) 可控制 VSI 層次的資料流量
* 設定子網路存取網際網路的公用閘道，由 ACL 進行保護
* 設定 VSI 存取網際網路的浮動 IP，由 SG 進行保護

![IBM VPC 連線功能及安全](images/vpc-connectivity-and-security.svg "IBM VPC 連線功能及安全"){: caption="圖：安全群組和 ACL 能為您的子網路及實例增加安全。" caption-side="top"}

## 定義
{: #definitions}

此[名詞解釋](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary)提供 ACL 及 SG 的定義和說明，以及您可以對其執行的動作。下一節說明 ACL 及安全群組的基本功能，以及 VPC 支援端對端加密的方式。

### 存取控制清單
{: #access-control-list}

**存取控制清單 (ACL)** 可以管理（亦即，容許或拒絕）子網路的入埠及出埠資料流量。ACL 無狀態，這表示必須個別且明確地指定入埠和出埠規則。每一個 ACL 都由基於*來源 IP*、*來源埠*、*目的地 IP*、*目的地埠* 和*通訊協定* 的規則組成。

在 {{site.data.keyword.cloud}} VPC 中，每個子網路都有建立預設 ACL（容許入埠及出埠資料流量），但客戶可以建立自訂 ACL。任何時候都只能有一個 ACL 連接至一個子網路，但一個 ACL 可以連接至多個子網路。如需如何使用 ACL 的相關資訊，請參閱 [ACL 手冊](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)。

### 安全群組
{: #security-group}

**安全群組**可用來作為虛擬防火牆，以控制一個以上伺服器 (VSI) 的資料流量。安全群組是規則集合，其指定是否容許相關聯 VSI 的資料流量。

當客戶建立 VSI 時，客戶可以建立一個以上安全群組與該 VSI 的關聯。假設有正確的許可權，客戶可以使用 IBM 主控台、CLI 或 API 來修改安全群組規則。

如需如何建立使用安全群組的 VSI 以及更多安全群組功能的相關資訊，請參閱[安全群組手冊](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups)。

### 端對端加密
{: #end-to-end-encryption}

雖然 IBM Cloud VPC 不提供端對端加密，但它容許使用。例如，假設虛擬伺服器上有安全端點（例如，埠 443 上的 HTTPS 伺服器），您可以將浮動 IP 連接至該伺服器，然後就會對從用戶端到埠 443 之伺服器間的連線進行端對端加密。路徑中的任何內容都不會強制執行解密。

不過請注意，如果您使用不安全的通訊協定（例如，埠 80 上的 HTTP），則資料從頭到尾都會很清楚。
