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


# リモート FortiGate ピアへのセキュア接続の作成
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

この資料は、FortiGate 300C、ファームウェア・バージョン	v5.2.13、build762 (GA) に基づいています。

以下の手順の例では、{{site.data.keyword.cloud}} API または CLI を使用して仮想プライベート・クラウドを作成するという前提条件となる手順を省略しています。 詳しくは、[概要](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)および [API を使用した VPC のセットアップ](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)を参照してください。

## 手順の例
{: #fortigate-example-steps}

リモートの FortiGate ピアに接続するためのトポロジーは、2 つの VPC 間に VPN 接続を作成するのと似ています。 ただし、片側が FortiGate 300C ユニットに置き換わっている点が異なります。

![画像の説明をここに入力](./images/vpc-vpn-fg-figure.png)

### リモート Fortigate ピアとのセキュア接続を作成するには、以下のようにします
{: #to-create-a-secure-connection-with-the-remote-fortigate-peer}

**「VPN」\>「IPsec」\>「トンネル (Tunnels)」**に移動し、新しいカスタム・トンネルを作成するか、既存のトンネルを編集します。

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

FortiGate ユニットは、リモート VPN ピアから接続要求を受け取ると、IPsec フェーズ 1 のパラメーターを使用してセキュア接続を確立し、その VPN ピアを認証します。 その後、セキュリティー・ポリシーにより接続が許可されると、FortiGate ユニットは IPsec フェーズのパラメーターを使用してトンネルを確立し、IPsec セキュリティー・ポリシーを適用します。 鍵管理サービス、認証サービス、およびセキュリティー・サービスは、IKE プロトコルを使用して動的にネゴシエーションされます。

**これらの機能をサポートするために、以下の一般的な構成ステップを FortiGate ユニットで実行する必要があります。**

* フェーズ 1 のパラメーターを定義します。これは、FortiGate ユニットがリモート・ピアの認証とセキュア接続の確立を行うために必要です。

* フェーズ 2 のパラメーターを定義します。これは、FortiGate ユニットがリモート・ピアへの VPN トンネルを作成するために必要です。

* セキュリティー・ポリシーを作成します。これは、許可されるサービスと、IP 送信元アドレスと宛先アドレス間のトラフィックの許可方向を制御します。

デフォルトでは、フェーズ 1 とフェーズ 2 のいくつかの典型的なパラメーターが設定されています。

### IBM Cloud VPC VPN の構成
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

IBM Cloud VPC の VPN 機能に接続するには、以下の構成が推奨されます。

1. 認証で IKEv2 を選択します
2. フェーズ 1 プロポーザルで `DH-group 2` を有効にします
3. フェーズ 1 プロポーザルで `lifetime = 36000` を設定します
4. フェーズ 2 プロポーザルで PFS を無効にします
5. フェーズ 2 プロポーザルで `lifetime = 10800` を設定します
6. フェーズ 2 でサブネットの情報を入力します
7. その他のパラメーターはデフォルト値のままにします。

![画像の説明をここに入力](./images/vpc-vpn-fg-network.JPG)

### ネットワーク・パラメーター
{: #fortigate-network-parameters}

![画像の説明をここに入力](./images/vpc-vpn-fg-authentication.JPG)

### 認証パラメーター
{: #fortigate-authentication-parameters}

![画像の説明をここに入力](./images/vpc-vpn-fg-phase1.JPG)

### フェーズ 1 のパラメーター
{: #fortigate-phase-1-parameters}

![画像の説明をここに入力](./images/vpc-vpn-fg-phase2.JPG)

### フェーズ 2 のパラメーター
{: #fortigate-phase-2-parameters}

## ローカル IBM Cloud VPC とのセキュア接続を作成するには、以下のようにします
{: #fortigate-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

セキュア接続を作成するには、VPC 内で VPN 接続を作成します。これは、2 つの VPC の例に似ています。

* VPC サブネットに VPN ゲートウェイを作成し、VPC と FortiGate ユニットの間に VPN 接続を作成し、`local_cidrs` を VPC のサブネット値に設定し、`peer_cidrs` を FortiGate のサブネット値に設定します。

**注:** VPN ゲートウェイの作成中はゲートウェイの状況は `pending` と表示され、作成が完了すると状況が `available` になります。 作成にはしばらく時間がかかる場合があります。

![画像の説明をここに入力](images/vpc-vpn-fg-connection.png)

### セキュア接続の状況の確認
{: #fortigate-check-the-status-of-the-secure-connection}

IBM Cloud コンソールから接続の状況を確認できます。 また、VSI を使用してサイト間で `ping` を実行することもできます。

![画像の説明をここに入力](images/vpc-vpn-fg-status.JPG)
