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

# VPC での VPN の使用
{: #--using-vpn-with-your-vpc}
[comment]: # (リンクされたヘルプ・トピック)

{{site.data.keyword.cloud}} VPC の VPN サービスを使用すると、プライベート・ネットワークをセキュアな方法で接続できます。 VPC とオンプレミスのプライベート・ネットワークとの間、または別の VPC との間で、VPN を使用して IPsec サイト間トンネルをセットアップできます。

現行の {{site.data.keyword.cloud}} VPC リリースでは、ポリシー・ベースのルーティングのみがサポートされています。

## フィーチャー
{: #vpn-features}

* IKEv1 および IKEv2
* 認証アルゴリズム: `md5`、`sha1`、`sha256`
* 暗号化アルゴリズム: `3des`、`aes128`、`aes256`
* Diffie-Hellman (DH) グループ: 2、5、14
* IKE ネゴシエーション・モード: main
* IPSec 変換プロトコル: ESP
* IPSec カプセル化モード: tunnel
* Perfect Forward Secrecy (PFS)
* 非活動ピア検出
* ルーティング: ポリシー・ベース
* 認証モード: 事前共有鍵
* HA サポート (アクティブ/スタンバイ・モードのみ)

## 使用可能な API
{: #apis-available}

次のセクションでは、IBM Cloud VPC 環境で VPN に使用できる API について詳しく説明します。 詳しくは、[VPC REST API](https://{DomainName}/apidocs/vpc-on-classic#list-all-ike-policies) ページを参照してください。

### VPN ゲートウェイと VPN 接続
{: #vpn-gateways-and-vpn-connections}

| 説明 | API |
|----------------------------|-------------|
| VPN ゲートウェイの作成 | POST /vpn_gateways |
| 複数の VPN ゲートウェイの取得 | GET /vpn_gateways |
| 1 つの VPN ゲートウェイの取得 | GET /vpn_gateways/{id} |
| VPN ゲートウェイの削除 | DELETE /vpn_gateways/{id} |
| VPN ゲートウェイの更新 | PATCH /vpn_gateways/{id} |
| 新規 VPN 接続の作成 | POST /vpn_gateways/{vpn_gateway_id}/connections |
| 複数の VPN 接続の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections |
| 1 つの VPN 接続の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 接続の削除 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 接続の更新 | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 接続のすべてのローカル CIDR の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| VPN 接続からのローカル CIDR の削除 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続に特定のローカル CIDR が存在するかどうかの確認 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続のローカル CIDR の設定 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続のすべてのピア CIDR の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| VPN 接続からのピア CIDR の削除 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続に特定のピア CIDR が存在するかどうかの確認 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続のピア CIDR の設定 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE ポリシー
{: #ike-policies}

| 説明 | API |
|-----------------------------|--------------|
| すべての IKE ポリシーの取得 | GET /ike_policies |
| IKE ポリシーの作成 | POST /ike_policies |
| IKE ポリシーの削除 | DELETE /ike_policies/{id} |
| IKE ポリシーの取得 | GET /ike_policies/{id} |
| IKE ポリシーの更新 | PATCH /ike_policies/{id} |
| 指定された IKE ポリシーを使用するすべての接続の取得 | GET /ike_policies/{id}/connections |

### IPsec ポリシー
{: #ipsec-policies}

| 説明 | API |
|---------------------|-------------|
| すべての IPSec ポリシーの取得 | GET /ipsec_policies |
| IPSec ポリシーの作成 | POST /ipsec_policies |
| IPSec ポリシーの削除 | DELETE /ipsec_policies/{id} |
| IPSec ポリシーの取得 | GET /ipsec_policies/{id} |
| IPSec ポリシーの更新 | PATCH /ipsec_policies/{id} |
| 指定された IPSec ポリシーを使用するすべての接続の取得 | GET /ipsec_policies/{id}/connections |

## VPN の例
{: #vpn-example}

以下の例では、VPN を使用して 2 つの VPC を接続できます。つまり、2 つの別個の VPC 内のサブネットを、単一のネットワークであるかのように接続できます。 各サブネットの IP アドレスがオーバーラップしてはなりません。
シナリオは以下のようになります (各 VPC にいくつかの VM が追加されています):

![IBM VPC のための VPN](images/vpc-vpn.svg "IBM VPC のための VPN")

### 手順の例
{: #vpn-example-steps}

以下の手順の例では、IBM Cloud API または CLI を使用して VPC を作成するという前提条件となる手順を省略しています。 詳しくは、[概要](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)および [API を使用した VPC のセットアップ](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)を参照してください。

オプションで、UI を使用して VPN ゲートウェイを作成できます。 この手順については、[コンソール・チュートリアルの資料](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)を参照してください。

#### 手順 1. VPC サブネットに VPN ゲートウェイを作成する
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

サンプル出力:
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

後続のステップのために次のフィールドが保存されていることを確認します。
* `id`。 これが VPN ゲートウェイの ID です。これ以降は、`$gwid1` と表すことにします。
* `address`。 これが VPN ゲートウェイのパブリック IP アドレスです。これ以降は、`$gwaddress1` として参照します。

VPN ゲートウェイの作成中はゲートウェイの状況が `pending` と表示されます。作成が完了すると状況が `available` になります。 作成にはしばらく時間がかかる場合があります。
{: note}


次のコマンドにより、ゲートウェイの状況を確認できます。

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### 手順 2. 2 番目の VPN ゲートウェイを別の VPC に作成する
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

サンプル出力:
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

後続のステップのために次のフィールドを必ず保存してください。
* `id`。 これが VPN ゲートウェイの ID です。これ以降は、`$gwid2` として参照します。
* `address`。 これが VPN ゲートウェイのパブリック IP アドレスです。これ以降は、`$gwaddress2` として参照します。


#### 手順 3. 1 番目の VPN ゲートウェイから 2 番目の VPN ゲートウェイへの VPN 接続を作成する
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

この接続を作成するときには、`local_cidrs` に **VPC 1** のサブネットを設定し、`peer_cidrs` に **VPC 2** のサブネットを設定します。

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

サンプル出力:
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

#### 手順 4. 2 番目の VPN ゲートウェイから 1 番目の VPN ゲートウェイへの VPN 接続を作成する
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

この接続を作成するときには、`local_cidrs` に **VPC 2** のサブネットを設定し、`peer_cidrs` に **VPC 1** のサブネットを設定します。

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

サンプル出力:
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

#### 手順 5. 接続を確認する
{: #step-5-verify-connectivity}

VPN 接続が確立されたら、サブネット 1 からサブネット 2 のインスタンスにアクセスできます (逆方向も可能です)。

VPN 接続の状況は、以下のようにして確認できます。
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

サンプル出力:
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

## 割り当て量
{: #see-vpn-quotas}

VPN 割り当て量については、[VPC 割り当て量](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas)のトピックを参照してください。

## ポリシーのオートネゴシエーション
{: #policy-auto-negotiation}

VPN 接続の構成に IKE ポリシーと IPsec ポリシーを使用することはオプションです。ポリシーが選択されていなければ、_オートネゴシエーション_ というデフォルトのプロポーザルがプロセスに対して自動的に選択されます。  

IBM Cloud のオートネゴシエーションでは **IKEv2** が使用されるため、オンプレミス・デバイスでも **IKEv2** を使用する必要があります。オンプレミス・デバイスが **IKEv2** をサポートしていない場合は、カスタマイズされた IKE ポリシーを使用してください。
{: note}

### IKE オートネゴシエーション (フェーズ 1)
{: #ike-auto-negotiation-phase-1}

次に示す暗号化、認証、および Diffie-Hellman グループのオプションは、自由に組み合わせて使用できます。

|    | 暗号化 | 認証 | DH グループ |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### IPsec オートネゴシエーション (フェーズ 2)
{: #ipsec-auto-negotiation-phase-2}

次に示す暗号化と認証のオプションは、自由に組み合わせて使用できます。

|    | 暗号化 | 認証 | DH グループ |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabled  |
| 2  | aes256 | sha256 | disabled  |
| 3  | 3des   | md5    | disabled  |

## FAQ
{: #vpn-faq}

**VPN ゲートウェイを作成するときに、VPN 接続を同時に作成できますか?**

API または CLI を使用する場合は、VPN ゲートウェイを作成した後で VPN 接続を作成する必要があります。 IBM Cloud コンソールでは、ゲートウェイと接続を同時に作成できます。

**VPN 接続が接続されている VPN ゲートウェイを削除した場合、接続はどうなりますか?**

VPN 接続は VPN ゲートウェイと共に削除されます。

**VPN ゲートウェイまたは VPN 接続を削除した場合、IKE ポリシーまたは IPSec ポリシーは削除されますか?**

削除されません。IKE ポリシーと IPSec ポリシーは複数の接続に適用できます。

**VPN ゲートウェイが配置されているサブネットを削除すると、そのゲートウェイはどうなりますか?**

インスタンス (VPN ゲートウェイを含む) が存在するサブネットは削除できません。

**デフォルトの IKE ポリシーと IPsec ポリシーはありますか?**

ポリシー ID (IKE または IPsec) を指定せずに VPN 接続を作成すると、オートネゴシエーションが使用されます。

**VPN ゲートウェイのプロビジョニング中にサブネットを選択する必要があるのはなぜですか?**

そのサブネットによって、VPN ゲートウェイは VPC 内の他のリソースに接続します。お勧めするベスト・プラクティスは、VPN ゲートウェイ専用のサブネット (そのサブネットには他の VPC インスタンスは存在しません) を作成し、そのサブネット内に十分な数の空きプライベート IP を確保することです。HA およびローリング・アップグレードに対応するためには、VPN ゲートウェイに 8 個のプライベート IP アドレスが必要になります。

**VPN ゲートウェイはどのゾーンに存在しますか?**

VPN ゲートウェイは、プロビジョニング中に選択したサブネットのゾーンにあります。VPN ゲートウェイは、同じゾーンおよび同じ VPC 内の VPC インスタンスのみを処理することを忘れないでください。したがって、他のゾーンの VPC インスタンスが、VPN ゲートウェイを利用してオンプレミスのプライベート・ネットワークと通信することはできません。ゾーン障害に対する耐障害性のためには、VPN ゲートウェイをゾーンごとに 1 つデプロイする必要があります。

**VPN ゲートウェイのデプロイに使用するサブネットで ACL を使用する場合は、何をする必要がありますか?**

管理トラフィックおよび VPN トンネル・トラフィックを許可するために、必ず、以下の ACL ルールを設定してください。

* **インバウンド・ルール**
    - プロトコル TCP ソース・ポート 9091 を許可
    - プロトコル TCP ソース・ポート 10514 を許可
    - プロトコル TCP ソース・ポート 443 を許可
    - プロトコル TCP ソース・ポート 80 を許可
    - プロトコル TCP ソース・ポート 53 を許可
    - プロトコル UDP ソース・ポート 53 を許可
    - プロトコル ALL、ソース IP「VPN ピア・ゲートウェイのパブリック IP」を許可
    - プロトコル TCP 宛先ポート 443 を許可
    - プロトコル TCP 宛先ポート 56500 を許可
    - VPC 内のインスタンスとオンプレミスのプライベート・ネットワークの間のトラフィックを許可
    - ICMP トラフィックを許可

* **アウトバウンド・ルール**
   - すべてのトラフィックを許可

**オンプレミスのプライベート・ネットワークと通信する必要のあるサブネットで ACL またはセキュリティー・グループを使用する場合は、何をする必要がありますか?**

VPC 内のインスタンスとオンプレミス・プライベート・ネットワークの間のトラフィックを許可する ACL ルールまたはセキュリティー・グループ・ルールを設定する必要があります。

**VPN for VPC では HA 構成がサポートされますか?**

はい。アクティブ/スタンバイ構成の HA がサポートされます。

**SSL VPN を利用できるプランはありますか?**

いいえ。IPsec サイト間のみがサポートされています。

**サイト間 VPNaaS のスループットに上限はありますか?**

最大 650 Mbps のスループットがサポートされます。

**VPNaaS で PSK および証明書ベースの IKE 認証はサポートされていますか?**

PSK 認証のみがサポートされています。

**VPN for VPC を IBM Cloud Infrastructure Classic の VPN ゲートウェイとして使用できますか?**

いいえ。 IBM Cloud Infrastructure Classic 環境で VPN ゲートウェイを使用するには、[IPsec VPN](https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn) を使用する必要があります。
