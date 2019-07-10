---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, FortiGate, connection, secure

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 建立與遠端 FortiGate 對等節點的安全連線
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

本文件係根據 FortiGate 300C 韌體版本 5.2.13 版 build762 (GA)。

接下來的範例步驟會跳過使用 {{site.data.keyword.cloud}} API 或 CLI 來建立 Virtual Private Cloud 的必要步驟。如需相關資訊，請參閱[開始使用](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)及[使用 API 的 VPC 設定](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 範例步驟
{: #fortigate-example-steps}

連接至遠端 FortiGate 對等節點的拓蹼類似於在兩個 VPC 之間建立 VPN 連線。不過，有一方是由 FortiGate 300C 裝置取代。

![在這裡輸入影像說明](./images/vpc-vpn-fg-figure.png)

### 建立與遠端 Fortigate 對等節點的安全連線
{: #to-create-a-secure-connection-with-the-remote-fortigate-peer}

移至 **VPN \> IPsec \> 通道**，然後建立新的自訂通道或編輯現有通道。

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

當 FortiGate 裝置接收到來自遠端 VPN 對等節點的連線要求時，它會使用 IPsec「階段 1」參數來建立安全連線，並鑑別該 VPN 對等節點。然後，如果安全原則允許連線，FortiGate 裝置會使用 IPsec 階段參數來建立通道，並套用 IPsec 安全原則。金鑰管理、鑑別及安全服務會透過 IKE 通訊協定動態協議。

**若要支援這些功能，FortiGate 裝置必須執行下列一般配置步驟：**

* 定義 FortiGate 裝置在鑑別遠端對等節點及建立安全連線時所需的「階段 1」參數。

* 定義 FortiGate 裝置在建立與遠端對等節點的 VPN 通道時所需的「階段 2」參數。

* 建立安全原則來控制所允許的服務，以及 IP 來源與目的地位址之間的資料流量的允許方向。

依預設，已設定某些一般「階段 1」和「階段 2」參數。

### IBM Cloud VPC VPN 的配置
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

若要連接至 IBM Cloud VPC 的 VPN 功能，建議您進行下列配置：

1. 在鑑別中選擇 IKEv2
2. 在「階段 1」提案中啟用 `DH-group 2`
3. 在「階段 1」提案中設定 `lifetime = 36000`
4. 在「階段 2」提案中停用 PFS
5. 在「階段 2」提案中設定 `lifetime = 10800`
6. 在「階段 2」中輸入子網路的資訊
7. 其餘參數會保留其預設值。

![在這裡輸入影像說明](./images/vpc-vpn-fg-network.JPG)

### 網路參數
{: #fortigate-network-parameters}

![在這裡輸入影像說明](./images/vpc-vpn-fg-authentication.JPG)

### 鑑別參數
{: #fortigate-authentication-parameters}

![在這裡輸入影像說明](./images/vpc-vpn-fg-phase1.JPG)

### 階段 1 參數
{: #fortigate-phase-1-parameters}

![在這裡輸入影像說明](./images/vpc-vpn-fg-phase2.JPG)

### 階段 2 參數
{: #fortigate-phase-2-parameters}

## 建立與本端 IBM Cloud VPC 的安全連線
{: #fortigate-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

若要建立安全連線，您要在 VPC 內建立 VPN 連線，其類似於 2 VPC 範例。

* 在 VPC 子網路上建立 VPN 閘道，以及在 VPC 與 FortiGate 裝置之間建立 VPN 連線，將 `local_cidrs` 設為 VPC 上的子網路值，並將 `peer_cidrs` 設為 FortiGate 上的子網路值。

**附註：**建立 VPN 閘道時，閘道狀態會顯示為 `pending`，建立完成之後，狀態會變成 `available`。建立作業可能需要一些時間。

![在這裡輸入影像說明](images/vpc-vpn-fg-connection.png)

### 檢查安全連線的狀態
{: #fortigate-check-the-status-of-the-secure-connection}

您可以透過 IBM Cloud 主控台檢查連線狀態。您也可以嘗試使用 VSI 執行點對點 `ping`。

![在這裡輸入影像說明](images/vpc-vpn-fg-status.JPG)
