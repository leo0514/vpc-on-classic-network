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

# リモート Cisco ASAv ピアとのセキュア接続の作成
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

この資料は、Cisco ASAv、Cisco Adaptive Security Appliance Software バージョン 9.10(1) に基づいています。

以下の手順の例では、{{site.data.keyword.cloud}} API または CLI を使用して仮想プライベート・クラウドを作成するという前提条件となる手順を省略しています。 詳しくは、[概要](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)および [API を使用した VPC のセットアップ](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)を参照してください。

## 手順の例
{: #cisco-example-steps}

リモートの Cisco ASAv ピアに接続するためのトポロジーは、2 つの {{site.data.keyword.cloud}} 仮想プライベート・クラウド間に VPN 接続を作成するのと似ています。 ただし、片側が Cisco ASAv ユニットに置き換わっている点が異なります。

![画像の説明をここに入力](./images/vpc-vpn-asav-figure.png)

### リモート Cisco ASAv ピアとのセキュア接続を作成するには、以下のようにします
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

IBM VPC の VPN で使用するための Cisco ASA の構成手順では、最初に以下の前提条件が満たされていることを確認します。

* Cisco ASAv がオンラインであり、適切なライセンスで機能している
* Cisco ASAv のパスワードが有効になっている
* 少なくとも 1 つの機能する内部インターフェースが構成され、検証されている
* 少なくとも 1 つの外部インターフェースが機能するように構成され、検証されている

Cisco ASAv ユニットは、リモート VPN ピアから接続要求を受け取ると、IPsec フェーズ 1 のパラメーターを使用してセキュア接続を確立し、その VPN ピアを認証します。 その後、セキュリティー・ポリシーにより接続が許可されると、Cisco ASAv は IPsec フェーズ 1 のパラメーターを使用してトンネルを確立し、IPsec セキュリティー・ポリシーを適用します。 鍵管理サービス、認証サービス、およびセキュリティー・サービスは、IKE プロトコルを使用して動的にネゴシエーションされます。

**これらの機能をサポートするために、以下の一般的な構成ステップを Cisco ASAv ユニットで実行する必要があります。**

* フェーズ 1 のパラメーターを定義します。これは、Cisco ASAv ユニットがリモート・ピアの認証とセキュア接続の確立を行うために必要です。
* フェーズ 2 のパラメーターを定義します。これは、Cisco ASAv ユニットがリモート・ピアへの VPN トンネルを作成するために必要です。

Internet Key Exchange (IKE) バージョン 2 プロポーザル・オブジェクトを作成します。 IKEv2 プロポーザル・オブジェクトには、
リモート・アクセスおよびサイト間 VPN ポリシーを定義する際に IKEv2 プロポーザルを作成するために必要なパラメーターが含まれています。 IKE は、IPsec ベースの通信の管理を容易にする鍵管理プロトコルです。 IPsec ピアの認証、IPsec 暗号鍵のネゴシエーションと配布、および IPsec セキュリティー・アソシエーション (SA) の自動確立に使用されます。 

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

IPsec 接続の IKEv2 ポリシー構成を作成します。 IKEv2 ポリシー・ブロックでは、IKE 交換のパラメーターが設定されます。 このブロックでは、次のパラメーターが設定されます。
* 暗号化アルゴリズム - この例では AES-256 に設定されます
* 整合性アルゴリズム - この例では SHA256 に設定されます
* Diffie-Hellman グループ - IPsec はピア間の初期暗号鍵の生成に Diffie-Hellman アルゴリズムを使用します。 この例では、グループ 14 に設定されます
* 疑似乱数関数 (PRF) - IKEv2 では、IKEv2 トンネル暗号化に必要な鍵マテリアルとハッシュ操作を導出するアルゴリズムとして、
別個の方式を使用する必要があります。 これは疑似乱数関数と呼ばれ、SHA に設定されます
* SA 存続時間 - セキュリティー・アソシエーションの存続時間を設定します (この時間の経過後に再接続が行われます)。 36,000 秒に設定されます。
* 操作タイプ - これはデフォルト値 bi-directional のままにします。 (「show running」表示で明示的に示されません。)

次のサンプル・コードに示すように、このサンプル・ポリシーは AES-256 を使用してセキュア・チャネルを暗号化します。 リモート・ピアの同一性の検証には SHA512 ハッシュ・アルゴリズムが使用され、
鍵の生成には Diffie-Hellman グループ 14 が使用されます。 グループ 14 は 2048 ビット暗号化ブロックを使用します。 最後に、セキュリティー・アソシエーションの存続期間が 36,000 秒に設定されます。

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* VPN のアクセス・リストと暗号マップを定義します。

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## ローカル IBM Cloud VPC とのセキュア接続を作成するには、以下のようにします
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

セキュア接続を作成するには、VPC 内で VPN 接続を作成します。これは、2 つの VPC の例に似ています。

* VPC サブネットに VPN ゲートウェイを作成し、VPC と Cisco ASAv の間に VPN 接続を作成し、`local_cidrs` を VPC のサブネット値に設定し、`peer_cidrs` を Cisco ASAv のサブネット値に設定します。

VPN ゲートウェイの作成中はゲートウェイの状況が `pending` と表示されます。作成が完了すると状況が `available` になります。 作成にはしばらく時間がかかる場合があります。 
{:note}


![画像の説明をここに入力](./images/vpc-vpn-asav-connection.png)

### セキュア接続の状況の確認
{: #cisco-check-the-status-of-the-secure-connection}

IBM Cloud コンソールから接続の状況を確認できます。 また、VSI を使用してサイト間で `ping` を実行することもできます。

![画像の説明をここに入力](./images/vpc-vpn-asav-status.png)
