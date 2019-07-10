---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# 建立與遠端 Cisco ASAv 對等節點的安全連線
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

本文件以 Cisco ASAv、Cisco Adaptive Security Appliance 軟體版本 9.10(1) 為基礎。

接下來的範例步驟會跳過使用 {{site.data.keyword.cloud}} API 或 CLI 來建立 Virtual Private Cloud 的必要步驟。如需相關資訊，請參閱[開始使用](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)及[使用 API 的 VPC 設定](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 範例步驟
{: #cisco-example-steps}

連接至遠端 Cisco ASAv 對等節點的拓蹼類似於在兩個 {{site.data.keyword.cloud}} Virtual Private Cloud 之間建立 VPN 連線。不過，有一方是由 Cisco ASAv 裝置取代。

![在這裡輸入影像說明](./images/vpc-vpn-asav-figure.png)

### 建立與遠端 Cisco ASAv 對等節點的安全連線
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

配置 Cisco ASA 與 IBM VPC VPN 搭配使用的首要步驟，是要確定已具備下列必條件：

* Cisco ASAv 在線上且正常運作，並有適當的授權
* 已啟用 Cisco ASAv 的密碼
* 至少有一個已配置且已驗證的正常運作內部介面
* 至少有一個已配置且已驗證的正常運作外部介面

當 Cisco ASAv 裝置接收到來自遠端 VPN 對等節點的連線要求時，它會使用 IPsec「階段 1」參數來建立安全連線，並鑑別該 VPN 對等節點。然後，如果安全原則允許連線，Cisco ASAv 裝置會使用 IPsec「階段 1」參數來建立通道，並套用 IPsec 安全原則。金鑰管理、鑑別及安全服務會透過 IKE 通訊協定動態協議。

**若要支援這些功能，Cisco ASAv 裝置必須執行下列一般配置步驟：**

* 定義 Cisco ASAv 裝置在鑑別遠端對等節點及建立安全連線時所需的「階段 1」參數。
* 定義 Cisco ASAv 裝置在建立與遠端對等節點的 VPN 通道時所需的「階段 2」參數。

建立「網際網路金鑰交換 (IKE)」第 2 版提案物件。IKEv2 提案物件包含在定義遠端存取及點對點 VPN 原則時，建立 IKEv2 提案所需的參數。IKE 是一種金鑰管理通訊協定，可協助管理 IPsec 型通訊。它是用來鑑別 IPsec 對等節點，協議及配送
IPsec 加密金鑰，並自動建立 IPsec 安全關聯 (SA)。 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2 
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <key value>
 ikev2 remote-authentication pre-shared-key <key value>
 ikev2 local-authentication pre-shared-key <key value>
```

建立 IPsec 連線的 IKEv2 原則配置。IKEv2 原則區塊會設定 IKE 交換的參數。在此區塊中，設定了下列參數：
* 加密演算法 - 此範例設為 AES-256
* 完整性演算法 - 此範例設為 SHA256
* Diffie-Hellman 群組 - IPsec 使用 Diffie-Hellman 演算法，在對等節點之間產生起始加密金鑰。在此範例中，它設為群組 14
* Pseudo-Random 函數 (PRF) - IKEv2 需要使用獨立的方法作為演算法，以衍生 IKEv2 通道加密所需的建鑰資料及雜湊作業。這就是所謂的虛擬亂數函數，且設為 SHA
* SA 生命期限 - 設定安全關聯的生命期限（在此時間之後就會重新連線）。設定為 36,000 秒。
* 作業類型 - 將此項目保留為預設值：雙向。（它在「顯示執行中」畫面上並不明確。）

如下列程式碼範例所示，這個範例原則使用 AES-256 將安全通道加密。SHA512 雜湊演算法是用來驗證遠端對等節點的身分，Diffie-Hellman 群組 14 是用來產生金鑰。群組 14 使用 2048 位元加密區塊。最後，安全關聯的生命期限是設為 36,000 秒。

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* 定義 VPN 的存取清單和加密對映：

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## 建立與本端 IBM Cloud VPC 的安全連線
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

若要建立安全連線，您要在 VPC 內建立 VPN 連線，其類似於 2 VPC 範例。

* 在 VPC 子網路上建立 VPN 閘道，以及在 VPC 與 Cisco ASAv 之間建立 VPN 連線，將 `local_cidrs` 設為 VPC 上的子網路值，並將 `peer_cidrs` 設為 Cisco ASAv 上的子網路值。

建立 VPN 閘道時，閘道狀態會顯示為 `pending`，建立完成之後，狀態會變成 `available`。建立作業可能需要一些時間。
{:note}


![在這裡輸入影像說明](./images/vpc-vpn-asav-connection.png)

### 檢查安全連線的狀態
{: #cisco-check-the-status-of-the-secure-connection}

您可以透過 IBM Cloud 主控台檢查連線狀態。您也可以嘗試使用 VSI 執行點對點 `ping`。

![在這裡輸入影像說明](./images/vpc-vpn-asav-status.png)
