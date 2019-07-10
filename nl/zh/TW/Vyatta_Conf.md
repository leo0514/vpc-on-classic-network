---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Vyatta, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 建立與遠端 Vyatta 對等節點的安全連線
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

本文件是以 Vyatta 版本為基礎：AT&T vRouter 5600 1801d。

接下來的範例步驟會跳過使用 {{site.data.keyword.cloud}} API 或 CLI 來建立 Virtual Private Cloud 的必要步驟。如需相關資訊，請參閱[開始使用](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)及[使用 API 的 VPC 設定](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 範例步驟
{: #vyatta-example-steps}

連接至遠端 Vyatta 對等節點的拓蹼類似於在兩個 {{site.data.keyword.cloud_notm}} VPC 之間建立 VPN 連線。不過，有一方是由 Vyatta 裝置取代。

![在這裡輸入影像說明](images/vpc-vpn-vy-figure.png)

### 建立與遠端 Vyatta 對等節點的安全連線
{: #vyatta-to-create-a-secure-connection-with-the-remote-vyatta-peer}

當 VPN 對等節點接收到來自遠端 VPN 對等節點的連線要求時，它會使用 IPsec「階段 1」參數來建立安全連線，並鑑別該 VPN 對等節點。然後，如果安全原則允許連線，Vyatta 裝置會使用 IPsec「階段 2」參數來建立通道，並套用 IPsec 安全原則。金鑰管理、鑑別及安全服務會透過 IKE 通訊協定動態協議。

您可以移至 Vyatta 指令行來配置 IPsec 通道，也可以上傳 `*.vcli` 檔案來載入您的配置。

**若要支援這些功能，Vyatta 裝置必須執行下列一般配置步驟：**

* 定義 Vyatta 裝置在鑑別遠端對等節點及建立安全連線時所需的「階段 1」參數。

* 定義 Vyatta 裝置在建立與遠端對等節點的 VPN 通道時所需的「階段 2」參數。

若要連接至 IBM Cloud VPC 的 VPN 功能，建議您進行下列配置：

1. 在鑑別中選擇 `IKEv2`
2. 在「階段 1」提案中啟用 `DH-group 2`
3. 在「階段 1」提案中設定 `lifetime = 36000`
4. 在「階段 2」提案中停用 PFS
5. 在「階段 2」提案中設定 `lifetime = 10800`
6. 在「階段 2」中輸入對等節點及子網路的資訊

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.161.151_test_ike
set security vpn ipsec ike-group 169.61.161.151_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.161.151_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.161.151_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.161.151_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.161.151_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.161.151_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.161.151 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.161.151 authentication pre-shared-secret ******
set security vpn ipsec site-to-site peer 169.61.161.151 ike-group 169.61.161.151_test_ike
set security vpn ipsec site-to-site peer 169.61.161.151 default-esp-group 169.61.161.151_test_ipsec
set security vpn ipsec site-to-site peer 169.61.161.151 description "automation test"
set security vpn ipsec site-to-site peer 169.61.161.151 local-address 169.45.74.119
set security vpn ipsec site-to-site peer 169.61.161.151 connection-type initiate


set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 local prefix 192.168.200.0/24
set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 remote prefix 192.168.17.0/28

commit
```
{: screen}

然後，您可以使用 SCP 將這個 `*.vcli` 檔案上傳至 Vyatta，以套用配置。

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### 建立與本端 IBM Cloud VPC 的安全連線
{: #vyatta-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

 若要建立安全連線，您要在 VPC 內建立 VPN 連線，其類似於 2 VPC 範例。

* 在 VPC 子網路上建立 VPN 閘道，以及在 VPC 與 Vyatta 裝置之間建立 VPN 連線，將 `local_cidrs` 設為 VPC 上的子網路值，並將 `peer_cidrs` 設為 Vyatta 裝置上的子網路值。

建立 VPN 閘道時，閘道狀態會顯示為 `pending`，建立完成之後，狀態會變成 `available`。建立作業可能需要一些時間。
{: note}

![在這裡輸入影像說明](images/vpc-vpn-vy-connection.png)

### 檢查安全連線的狀態
{: #vyatta-check-the-status-of-the-secure-connection}

您可以透過 {{site.data.keyword.cloud_notm}} 主控台檢查連線狀態。您也可以嘗試使用 VSI 執行點對點 `ping`。

![在這裡輸入影像說明](images/vpc-vpn-vy-status.png)
