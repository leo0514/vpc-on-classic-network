---

copyright:
  years: 2019

lastupdated: "2019-05-20"

keywords: security groups, traffic, firewall, stateful, filtering

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# セキュリティー・グループの使用
{: #using-security-groups}
[comment]: # (リンクされたヘルプ・トピック)

セキュリティー・グループを使用すると、仮想サーバー・インスタンス (VSI) の各ネットワーク・インターフェースに、IP アドレスに基づいてフィルターを設定するルールを簡単に適用できます。 セキュリティー・グループのリソースを新規作成するときには、必要なネットワーク・トラフィック・パターンを作成するためにセキュリティー・グループを更新します。

セキュリティー・グループと ACL の特性の比較については、[比較表](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists)を参照してください。

デフォルトでは、セキュリティー・グループはすべてのトラフィックを拒否します。 ルールがセキュリティー・グループに追加されると、セキュリティー・グループが許可するトラフィックが定義されます。

ルールは_ステートフル_ です。つまり、許可されるトラフィックの応答として出される逆方向のトラフィックが自動的に許可されます。 例えば、ポート 80 でインバウンド TCP トラフィックを許可するルールは、ポート 80 で発信元ホストへのアウトバウンド TCP トラフィックの返信も許可します。このための追加ルールは不要です。

セキュリティー・グループの適用範囲は単一の VPC です。 適用範囲がこのようになっているので、セキュリティー・グループは同じ VPC 内に属する各 VSI のネットワーク・インターフェースに_のみ_ 付加することができます。

セキュリティー・グループを指定せずに VSI を作成すると、その VSI の 1 次ネットワーク・インターフェースは、その VSI の VPC の_デフォルト_ のセキュリティー・グループに置かれます。 この {{site.data.keyword.cloud}} VPC リリースでは、特定のトラフィックを許可するデフォルトのセキュリティー・グループが定義されています。 詳細については、[デフォルトのセキュリティー・グループの更新](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-updating-the-default-security-group)を参照してください。

以下のように、REST API、CLI、または UI を使用してセキュリティー・グループをセットアップできます。

* [API を使用したセキュリティー・グループのセットアップ](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-apis)
* [CLI を使用したセキュリティー・グループのセットアップ](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [UI を使用したセキュリティー・グループのセットアップ](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
