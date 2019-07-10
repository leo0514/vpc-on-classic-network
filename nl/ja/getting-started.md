---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-06-06"

keywords: provisioning, resources, permissions

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# 入門チュートリアル
{: #getting-started}

{{site.data.keyword.cloud}} 仮想プライベート・クラウドのネットワーキングを開始するには、以下のようにします。

1. 仮想プライベート・クラウドを作成します。
2. 1 つ以上のゾーンで仮想プライベート・クラウドに 1 つ以上のサブネットを作成します。
3. サブネット上のリソースがインターネットにアクセスできるようにするには (インターネットからのアクセスの場合も同様)、サブネット上にパブリック・ゲートウェイ (PGW) を作成します。

## 前提条件

 * **ユーザー許可**: VPC でリソースを作成および管理するための十分な許可がユーザーに付与されていることを確認してください。 必要な許可のリストについては、[VPC ユーザーに必要な許可の付与](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources)を参照してください。

## UI、CLI、または REST API を使用して VPC ネットワーク・リソースをプロビジョンする

UI、CLI、または REST API を使用して VPC ネットワーク・リソースのプロビジョンや管理を行うことができます。

* ユーザー・インターフェースでアクセスする場合は、[IBM Cloud コンソール ![外部リンク・アイコン](../../icons/launch-glyph.svg "外部リンク・アイコン")]( https://{DomainName}/vpc){: new_window} にログインします。 支援が必要な場合は、[UI ガイド](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console) に従ってください。
* コマンド・ライン・インターフェースを使用するには、[Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) の例に従います。
* 上級者ユーザーは、直接 [REST API](https://{DomainName}/apidocs/vpc-on-classic) を呼び出せます。 REST API を開始するには、[サンプル・コード](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)のチュートリアルに従ってください。

## 次のステップ

仮想プライベート・クラウドのネットワーキングをプロビジョンしたら、探索を進めてください。

* [仮想サーバー・インスタンスの作成および管理](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [セキュリティー・グループの使用](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [ネットワーク ACL の使用](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [VPN の使用](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [ロード・バランサーの使用](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
