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


# リモート StrongSwan ピアとのセキュア接続の作成
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

この資料は、Strongswan バージョン Linux StrongSwan U5.3.5/K4.4.0-133-generic に基づいています。

以下の手順の例では、{{site.data.keyword.cloud}} API または CLI を使用して仮想プライベート・クラウドを作成するという前提条件となる手順を省略しています。 詳しくは、[概要](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)および [API を使用した VPC のセットアップ](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)を参照してください。

## 手順の例
{: #strongswan-example-steps}

リモートの StrongSwan ピアに接続するためのトポロジーは、2 つの VPC 間に VPN 接続を作成するのと似ています。 ただし、接続の一方の側が StrongSwan ユニットに置き換わっている点が異なります。

![画像の説明をここに入力](./images/vpc-vpn-sw-figure.png)

### リモート StrongSwan ピアとのセキュア接続を作成するには、以下のようにします
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

**/etc** に移動し、新しいカスタム・トンネル構成ファイルを作成して **ipsec.abc.conf** などの名前を付けます。 **/etc/ipsec.conf** を編集し、次の行を追加して **ipsec.abc.conf** を組み込みます。

    include /etc/ipsec.abc.conf

VPN ピアは、リモート VPN ピアから接続要求を受け取ると、IPsec フェーズ 1 のパラメーターを使用してセキュア接続を確立し、その VPN ピアを認証します。 その後、セキュリティー・ポリシーにより接続が許可されると、StrongSwan ユニットは IPsec フェーズ 2 のパラメーターを使用してトンネルを確立し、IPsec セキュリティー・ポリシーを適用します。 鍵管理サービス、認証サービス、およびセキュリティー・サービスは、IKE プロトコルを使用して動的にネゴシエーションされます。

**これらの機能をサポートするために、以下の一般的な構成ステップを StrongSwan ユニットで実行する必要があります。**

* フェーズ 1 のパラメーターを定義します。これは、StrongSwan がリモート・ピアの認証とセキュア接続の確立を行うために必要です。

* フェーズ 2 のパラメーターを定義します。これは、StrongSwan がリモート・ピアへの VPN トンネルを作成するために必要です。
IBM Cloud VPC の VPN 機能に接続するには、以下の構成が推奨されます。

1. 認証で `IKEv2` を選択します
2. フェーズ 1 プロポーザルで `DH-group 2` を有効にします
3. フェーズ 1 プロポーザルで `lifetime = 36000` を設定します
4. フェーズ 2 プロポーザルで PFS を無効にします
5. フェーズ 2 プロポーザルで `lifetime = 10800` を設定します
6. フェーズ 2 プロポーザルでピアとサブネットの情報を入力します

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

`/etc/ipsec.secrets` で事前共有鍵を設定します

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

構成ファイルの実行が完了したら、StrongSwan ユニットを再始動します。

```
 ipsec restart
```
{: screen}

### ローカル IBM Cloud VPC とのセキュア接続を作成するには、以下のようにします
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* セキュア接続を作成するには、VPC 内で VPN 接続を作成します。これは、2 つの VPC の例に似ています。

* VPC サブネットに VPN ゲートウェイを作成し、VPC と StrongSwan の間に VPN 接続を作成し、`local_cidrs` を VPC のサブネット値に設定し、`peer_cidrs` を StrongSwan のサブネット値に設定します。

VPN ゲートウェイの作成中はゲートウェイの状況が `pending` と表示されます。作成が完了すると状況が `available` になります。 作成にはしばらく時間がかかる場合があります。
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### セキュア接続の状況の確認
{: #strongswan-check-the-status-for-a-secure-connection}

IBM Cloud コンソールから接続の状況を確認できます。 また、VSI を使用してサイト間で `ping` を実行することもできます。

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
