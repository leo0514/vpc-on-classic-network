---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{: note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 建立與遠端 StrongSwan 對等節點的安全連線
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

本文件以 Strongswan 為基礎，版本為 Linux StrongSwan U5.3.5/K4.4.0-133-generic。

接下來的範例步驟會跳過使用 {{site.data.keyword.cloud}} API 或 CLI 來建立 Virtual Private Cloud 的必要步驟。如需相關資訊，請參閱[開始使用](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)及[使用 API 的 VPC 設定](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 範例步驟
{: #strongswan-example-steps}

連接至遠端 StrongSwan 對等節點的拓蹼類似於在兩個 VPC 之間建立 VPN 連線。不過，連線有一方會由 StrongSwan 裝置取代。

![在這裡輸入影像說明](./images/vpc-vpn-sw-figure.png)

### 建立與遠端 StrongSwan 對等節點的安全連線
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

移至 **/etc**，然後以類似於 **ipsec.abc.conf** 的名稱建立新的自訂通道配置檔。編輯 **/etc/ipsec.conf**，新增下面這一行來包括 **ipsec.abc.conf**：

    include /etc/ipsec.abc.conf

當 VPN 對等節點接收到來自遠端 VPN 對等節點的連線要求時，它會使用 IPsec「階段 1」參數來建立安全連線，並鑑別該 VPN 對等節點。然後，如果安全原則允許連線，StrongSwan 裝置會使用 IPsec「階段 2」參數來建立通道，並套用 IPsec 安全原則。金鑰管理、鑑別及安全服務會透過 IKE 通訊協定動態協議。

**若要支援這些功能，StrongSwan 裝置必須執行下列一般配置步驟：**

* 定義 StrongSwan 在鑑別遠端對等節點及建立安全連線時所需的「階段 1」參數。

* 定義 StrongSwan 在建立與遠端對等節點的 VPN 通道時所需的「階段 2」參數。若要連接至 IBM Cloud VPC 的 VPN 功能，建議您進行下列配置：

1. 在鑑別中選擇 `IKEv2`
2. 在「階段 1」提案中啟用 `DH-group 2`
3. 在「階段 1」提案中設定 `lifetime = 36000`
4. 在「階段 2」提案中停用 PFS
5. 在「階段 2」提案中設定 `lifetime = 10800`
6. 在「階段 2」提案中輸入對等節點及子網路的資訊

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: screen}

在 `/etc/ipsec.secrets` 中設定預先共用金鑰

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

在配置檔完成執行之後，請重新啟動 StrongSwan 裝置。

```
 ipsec restart
```
{: screen}

### 建立與本端 IBM Cloud VPC 的安全連線
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* 若要建立安全連線，您要在 VPC 內建立 VPN 連線，其類似於 2 VPC 範例。

* 在 VPC 子網路上建立 VPN 閘道，以及在 VPC 與 StrongSwan 之間建立 VPN 連線，將 `local_cidrs` 設為 VPC 上的子網路值，並將 `peer_cidrs` 設為 StrongSwan 上的子網路值。

建立 VPN 閘道時，閘道狀態會顯示為 `pending`，建立完成之後，狀態會變成 `available`。建立作業可能需要一些時間。
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### 檢查安全連線的狀態
{: #strongswan-check-the-status-for-a-secure-connection}

您可以透過 IBM Cloud 主控台檢查連線狀態。您也可以嘗試使用 VSI 執行點對點 `ping`。

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
