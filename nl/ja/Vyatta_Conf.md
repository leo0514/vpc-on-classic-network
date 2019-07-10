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


# リモート Vyatta ピアとのセキュア接続の作成
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

この資料は、Vyatta バージョン: AT&T vRouter 5600 1801d に基づいています。

以下の手順の例では、{{site.data.keyword.cloud}} API または CLI を使用して仮想プライベート・クラウドを作成するという前提条件となる手順を省略しています。 詳しくは、[概要](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)および [API を使用した VPC のセットアップ](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)を参照してください。

## 手順の例
{: #vyatta-example-steps}

リモートの Vyatta ピアに接続するためのトポロジーは、2 つの {{site.data.keyword.cloud_notm}} VPC 間に VPN 接続を作成するのと似ています。 ただし、片側が Vyatta ユニットに置き換わっている点が異なります。

![画像の説明をここに入力](images/vpc-vpn-vy-figure.png)

### リモート Vyatta ピアとのセキュア接続を作成するには、以下のようにします
{: #vyatta-to-create-a-secure-connection-with-the-remote-vyatta-peer}

VPN ピアは、リモート VPN ピアから接続要求を受け取ると、IPsec フェーズ 1 のパラメーターを使用してセキュア接続を確立し、その VPN ピアを認証します。 その後、セキュリティー・ポリシーにより接続が許可されると、Vyatta ユニットは IPsec フェーズ 2 のパラメーターを使用してトンネルを確立し、IPsec セキュリティー・ポリシーを適用します。 鍵管理サービス、認証サービス、およびセキュリティー・サービスは、IKE プロトコルを使用して動的にネゴシエーションされます。

Vyatta コマンド・ラインに移動して IPsec トンネルを構成するか、`*.vcli` ファイルをアップロードして構成をロードすることができます。

**これらの機能をサポートするために、以下の一般的な構成ステップを Vyatta ユニットで実行する必要があります。**

* フェーズ 1 のパラメーターを定義します。これは、Vyatta ユニットがリモート・ピアの認証とセキュア接続の確立を行うために必要です。

* フェーズ 2 のパラメーターを定義します。これは、Vyatta ユニットがリモート・ピアへの VPN トンネルを作成するために必要です。

IBM Cloud VPC の VPN 機能に接続するには、以下の構成が推奨されます。

1. 認証で `IKEv2` を選択します
2. フェーズ 1 プロポーザルで `DH-group 2` を有効にします
3. フェーズ 1 プロポーザルで `lifetime = 36000` を設定します
4. フェーズ 2 プロポーザルで PFS を無効にします
5. フェーズ 2 プロポーザルで `lifetime = 10800` を設定します
6. フェーズ 2 でピアとサブネットの情報を入力します

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

次に、SCP を使用してこの `*.vcli` ファイルを Vyatta にアップロードし、構成を適用できます。

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### ローカル IBM Cloud VPC とのセキュア接続を作成するには、以下のようにします
{: #vyatta-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

 セキュア接続を作成するには、VPC 内で VPN 接続を作成します。これは、2 つの VPC の例に似ています。

* VPC サブネットに VPN ゲートウェイを作成し、VPC と Vyatta ユニットの間に VPN 接続を作成し、`local_cidrs` を VPC のサブネット値に設定し、`peer_cidrs` を Vyatta ユニットのサブネット値に設定します。

VPN ゲートウェイの作成中はゲートウェイの状況が `pending` と表示されます。作成が完了すると状況が `available` になります。 作成にはしばらく時間がかかる場合があります。
{: note}

![画像の説明をここに入力](images/vpc-vpn-vy-connection.png)

### セキュア接続の状況の確認
{: #vyatta-check-the-status-of-the-secure-connection}

{{site.data.keyword.cloud_notm}} コンソールから接続の状況を確認できます。また、VSI を使用してサイト間で `ping` を実行することもできます。

![画像の説明をここに入力](images/vpc-vpn-vy-status.png)
