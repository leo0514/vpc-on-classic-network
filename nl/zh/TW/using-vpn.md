---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-06-04"

keywords: VPN, network, encryption, authentication, algorithm, IKE, IPsec, policies, gateway, auto-negotiation

subcollection: vpc-on-classic-network


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# 使用 VPN 與 VPC 搭配
{: #--using-vpn-with-your-vpc}
[comment]: # (鏈結的說明主題)

{{site.data.keyword.cloud}} VPC VPN 服務可讓您以安全的方式連接專用網路。您可以使用 VPN 在您的 VPC 與內部部署的專用網路或另一個 VPC 之間設定 IPsec 端對端通道。

對於現行 {{site.data.keyword.cloud}} VPC 版本，僅支援原則型遞送。

## 特性
{: #vpn-features}

* IKEv1 和 IKEv2
* 鑑別演算法：`md5`、`sha1`、`sha256`
* 加密演算法：`3des`、`aes128`、`aes256`
* Diffie-Hellman (DH) 群組：2、5、14
* IKE 協議模式：main
* IPSec 轉換通訊協定：ESP
* IPSec 封裝模式：tunnel
* 完整轉遞保密 (PFS)
* 無回應對等節點偵測
* 遞送：原則型
* 鑑別模式：預先共用金鑰
* 僅限作用中/待命模式中的 HA 支援

## 可用的 API
{: #apis-available}

下一節提供有關您可在 IBM Cloud VPC 環境中用於 VPN 之 API 的詳細資料。如需詳細資料，請參閱 [VPC REST API](https://{DomainName}/apidocs/vpc-on-classic#list-all-ike-policies) 頁面。

### VPN 閘道及 VPN 連線
{: #vpn-gateways-and-vpn-connections}

| 說明 | API |
|----------------------------|-------------|
| 建立VPN 閘道 | POST /vpn_gateways |
| 擷取 VPN 閘道 | GET /vpn_gateways |
| 擷取 VPN 閘道 | GET /vpn_gateways/{id} |
| 刪除 VPN 閘道 | DELETE /vpn_gateways/{id} |
| 更新 VPN 閘道 | PATCH /vpn_gateways/{id} |
| 建立新的 VPN 連線 | POST /vpn_gateways/{vpn_gateway_id}/connections |
| 擷取 VPN 連線 | GET /vpn_gateways/{vpn_gateway_id}/connections |
| 擷取 VPN 連線 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| 刪除 VPN 連線 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| 更新 VPN 連線 | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| 擷取 VPN 連線的所有本端 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| 刪除 VPN 連線的本端 CIDR | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 檢查 VPN 連線上是否有特定的本端 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 在 VPN 連線上設定本端 CIDR | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 擷取 VPN 連線的所有對等節點 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| 刪除 VPN 連線的對等節點 CIDR | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| 檢查 VPN 連線上是否有特定的對等節點 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| 在 VPN 連線上設定對等節點 CIDR | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE 原則
{: #ike-policies}

| 說明 | API |
|-----------------------------|--------------|
| 擷取所有 IKE 原則 | GET /ike_policies |
| 建立 IKE 原則 | POST /ike_policies |
| 刪除 IKE 原則 | DELETE /ike_policies/{id} |
| 擷取 IKE 原則 | GET /ike_policies/{id} |
| 更新 IKE 原則 | PATCH /ike_policies/{id} |
| 擷取所有使用指定 IKE 原則的連線 | GET /ike_policies/{id}/connections |

### IPsec 原則
{: #ipsec-policies}

| 說明 | API |
|---------------------|-------------|
| 擷取所有 IPSec 原則 | GET /ipsec_policies |
| 建立 IPSec 原則 | POST /ipsec_policies |
| 刪除 IPSec 原則 | DELETE /ipsec_policies/{id} |
| 擷取 IPSec 原則 | GET /ipsec_policies/{id} |
| 更新 IPSec 原則 | PATCH /ipsec_policies/{id} |
| 擷取所有使用指定 IPsec 原則的連線 | GET /ipsec_policies/{id}/connections |

## VPN 範例
{: #vpn-example}

在下列範例中，您可以使用 VPN 連接兩個 VPC，這表示您可以將兩個不同 VPC 中的子網路連接，彷彿它們是一個單一網路。子網路的 IP 位址不得重疊。
情境範例如下（每個 VPC 新增一些 VM）：

![IBM VPC 的 VPN](images/vpc-vpn.svg "IBM VPC 的 VPN"){: caption="圖：IBM VPC 的 VPN" caption-side="top"}

### 範例步驟
{: #vpn-example-steps}

接下來的範例步驟會跳過使用 IBM Cloud API 或 CLI 來建立 VPC 的必要步驟。如需相關資訊，請參閱[開始使用](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)及[使用 API 的 VPC 設定](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

您也可以選擇使用使用者介面來建立 VPN 閘道。您可以在[主控台指導教學文件](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)中找到相關步驟。

#### 步驟 1. 在 VPC 子網路中建立 VPN 閘道
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

輸出範例：
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

請務必儲存下列欄位供後續步驟使用。
* `id`。這是 VPN 閘道 ID，將之稱為 `$gwid1`。
* `address`。這是 VPN 閘道的公用 IP 位址，將稱之為 `$gwaddress1`。

建立 VPN 閘道時，閘道狀態會顯示為 `pending`，建立完成之後，狀態會變成 `available`。建立作業可能需要一些時間。
{: note}


您可以使用下列指令來檢查閘道的狀態：

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### 步驟 2. 在不同的 VPC 上建立第二個 VPN 閘道
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

輸出範例：
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

請務必儲存下列欄位供後續步驟使用。
* `id`。這是 VPN 閘道 ID，將之稱為 `$gwid2`。
* `address`。這是 VPN 閘道的公用 IP 位址，將稱之為 `$gwaddress2`。


#### 步驟 3. 建立從第一個 VPN 閘道到第二個 VPN 閘道的 VPN 連線
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

當您建立連線時，請將 `local_cidrs` 設為**第一個 VPC** 的子網路，並將 `peer_cidrs` 設為**第二個 VPC** 的子網路。

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

輸出範例：
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
        "interval": 30,
        "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 步驟 4. 建立從第二個 VPN 閘道到第一個 VPN 閘道的 VPN 連線
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

當您建立連線時，請將 `local_cidrs` 設為**第二個 VPC** 的子網路，並將 `peer_cidrs` 設為**第一個 VPC** 的子網路。

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress1,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

輸出範例：
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
        "interval": 30,
        "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 步驟 5. 驗證連線功能
{: #step-5-verify-connectivity}

建立 VPN 連線之後，您就可以從第一個子網路連接第二個子網路上的實例，反之亦然。

您可以檢查 VPN 連線的狀態，如下所示：
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

輸出範例：
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## 配額
{: #see-vpn-quotas}

請參閱 [VPC 配額](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas)主題，以瞭解 VPN 配額。

## 原則自動協調
{: #policy-auto-negotiation}

可以選擇使用 IKE 及 IPsec 原則來配置 VPN 連線。若未選取任何原則，則會自動針對處理程序選擇預設提案，稱之為_自動協調_。 

IBM Cloud 會自動協調使用 **IKEv2**，因此，內部部署裝置也必須使用 **IKEv2**。如果您的內部部署裝置不支援 **IKEv2**，請使用自訂的 IKE 原則。
{: note}

### IKE 自動協調（階段 1）
{: #ike-auto-negotiation-phase-1}

可以任何組合來使用下列加密、鑑別及 Diffie-Hellman 群組選項：

|    | 加密 | 鑑別 | DH 群組 |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### IPsec 自動協調（階段 2）
{: #ipsec-auto-negotiation-phase-2}

可以任何組合來使用下列加密及鑑別選項：

|    | 加密 | 鑑別 | DH 群組 |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabled  |
| 2  | aes256 | sha256 | disabled  |
| 3  | 3des   | md5    | disabled  |

## 常見問題 (FAQ)
{: #vpn-faq}

**當我建立 VPN 閘道時，可以同時建立 VPN 連線嗎？**

如果您使用 API 或 CLI，則必須在建立 VPN 閘道之後建立 VPN 連線。在 IBM Cloud 主控台中，您可以同時建立閘道及連線。

**如果我刪除已連接 VPN 連線的 VPN 閘道，連線會發生什麼情況？**

VPN 連線會連同 VPN 閘道一併刪除。

**如果我刪除 VPN 閘道或 VPN 連線，也會一併刪除 IKE 或 IPSec 原則嗎？**

不，IKE 及 IPSec 原則可套用至多個連線。

**如果我嘗試刪除閘道所在的子網路時，VPN 閘道會發生什麼情況？**

如果有任何實例存在（包括 VPN 閘道在內），就無法刪除子網路。

**是否有預設 IKE 及 IPsec 原則？**

當您建立 VPN 連線而未參照原則 ID（IKE 或 IPsec）時，會使用自動協調。

**為何需要在 VPN 閘道佈建期間選擇子網路？**

子網路會將 VPN 閘道與 VPC 中的其他資源連接。建議的最佳作法是為 VPN 閘道建立專用子網路，且這個子網路上沒有其他 VPC 實例，以確保子網路中有足夠的可用專用 IP。VPN 閘道需要 8 個專用 IP 位址，才能容納 HA 及漸進式升級。

**VPN 閘道位於哪個區域中？**

VPN 閘道位於具有您在佈建期間所選擇子網路的區域中。請記住，VPN 閘道只會為相同區域及相同 VPC 中的 VPC 實例提供服務。因此，其他區域中的 VPC 實例無法利用 VPN 閘道與內部部署的專用網路進行通訊。針對區域容錯，您必須為每個區域部署一個 VPN 閘道。

**如果我要在用來部署 VPN 閘道的子網路上使用 ACL，該怎麼辦？**

請確定下列 ACL 規則已備妥，可容許管理資料流量及 VPN 通道資料流量：

* **入埠規則**
    - 容許通訊協定 TCP 來源埠 9091
    - 容許通訊協定 TCP 來源埠 10514
    - 容許通訊協定 TCP 來源埠 443
    - 容許通訊協定 TCP 來源埠 80
    - 容許通訊協定 TCP 來源埠 53
    - 容許通訊協定 UDP 來源埠 53
    - 容許通訊協定 ALL 來源 IP 為 VPN 同層級閘道公用 IP
    - 容許通訊協定 TCP 目的地埠 443
    - 容許通訊協定 TCP 目的地埠 56500
    - 容許 VPC 中的實例與內部部署專用網路之間的資料流量
    - 容許 ICMP 資料流量

* **出埠規則**
   - 容許所有資料流量

**如果我在需要與內部部署專用網路進行通訊的子網路上使用 ACL 或安全群組，該怎麼辦？**

您必須確定 ACL 規則或安全群組規則已備妥，可容許 VPC 中的實例與內部部署專用網路之間的資料流量。

**VPC 的 VPN 是否支援 HA 配置？**

是，它在作用中/待命配置中支援 HA。

**是否有支援 SSL VPN 的方案？**

沒有，僅支援 IPsec 點對點。

**點對點 VPNaaaS 是否有傳輸量上限？**

我們支援高達 650 Mbps 的傳輸量。

**VPNaaS 是否支援 PSK 及憑證型 IKE 鑑別？**

僅支援 PSK 鑑別。

**是否可以使用 VPC 的 VPN 作為 IBM Cloud Infrastructure Classic 的 VPN 閘道？**

不可以，為了在您的 IBM Cloud Infrastructure Classic 環境中使用「VPN 閘道」，您必須使用 [IPsec VPN](https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn)。
