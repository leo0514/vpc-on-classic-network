---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-29"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# IBM Cloud VPC のセキュリティー
{: #security-in-your-ibm-cloud-vpc}

ネットワーク・トラフィックの制御にセキュリティー・グループ (SG)、ネットワーク・アクセス制御リスト (ACL)、またはこの両方のタイプの制御を使用することで、VPC とワークロードを保護できます。ただし、次のことに注意してください。

* セキュリティー・グループはインスタンス (VSI) 単位でトラフィックを制御します。
* アクセス制御リストはサブネット単位でトラフィックを制御します。

## セキュリティーの概要
{: #security-overview}

* サブネットとの間のトラフィックは、アクセス制御リスト (ACL) で制御できます
* セキュリティー・グループ (SG) により、VSI レベルでトラフィックを制御できます
* サブネットからインターネットにアクセスするためのパブリック・ゲートウェイを、ACL で保護してセットアップします
* VSI からインターネットにアクセスするための浮動 IP を、SG で保護してセットアップします

![IBM VPC 接続性およびセキュリティー](images/vpc-connectivity-and-security.svg "IBM VPC 接続性およびセキュリティー")

## 定義
{: #definitions}

この[用語集](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary)には、ACL および SG の定義と説明、およびそれらを使用して実行できるアクションが記載されています。以下のセクションでは、ACL およびセキュリティー・グループの基本機能と、VPC でエンドツーエンド暗号化がサポートされる方法について説明します。

### アクセス制御リスト
{: #access-control-list}

**アクセス制御リスト (ACL)** は、サブネットのインバウンドとアウトバウンドのトラフィックを管理 (つまり許可または拒否) できます。 ACL はステートレスです。つまり、インバウンド・ルールおよびアウトバウンド・ルールは、別個に明示的に指定する必要があります。 各 ACL は、*送信元 IP*、*送信元ポート*、*宛先 IP*、*宛先ポート*、および*プロトコル* に基づいたルールで構成されます。

{{site.data.keyword.cloud}} VPC では、すべてのサブネットはデフォルト ACL を使用して作成されます。このデフォルト ACL は、インバウンドとアウトバウンドのトラフィックを許可しますが、お客様はカスタム ACL を作成できます。 サブネットには常に 1 つの ACL だけが付加されます。ただし 1 つの ACL を複数のサブネットに付加できます。 ACL の使用方法について詳しくは、[ACL ガイド](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)を参照してください。

### セキュリティー・グループ
{: #security-group}

**セキュリティー・グループ**は、1 つ以上のサーバー (VSI) のトラフィックを制御する仮想ファイアウォールとして機能します。 セキュリティー・グループは、関連付ける VSI に対してトラフィックを許可するかどうかを指定するルールの集合です。

お客様は VSI を作成したら、1 つ以上のセキュリティー・グループをその VSI に関連付けることができます。 正しい許可が付与されていれば、お客様は IBM Console、CLI、または API を使用してセキュリティー・グループ・ルールを変更できます。

セキュリティー・グループを使用する VSI の作成方法と、セキュリティー・グループの機能について詳しくは、[セキュリティー・グループ・ガイド](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups)を参照してください。

### エンドツーエンドの暗号化
{: #end-to-end-encryption}

IBM Cloud VPC はエンドツーエンドの暗号化を提供しませんが、許可はします。 例えば、仮想サーバーにセキュア・エンドポイント (ポート 443 での HTTPS サーバーなど) がある場合、そのサーバーに浮動 IP を付加することが可能で、ポート 443 でクライアントからサーバーの接続がエンドツーエンドで暗号化されます。  このパス内には復号を強制するものはありません。

ただし、非セキュア・プロトコル (ポート 80 での HTTP など) を使用すると、データはエンドツーエンドで平文になります。
