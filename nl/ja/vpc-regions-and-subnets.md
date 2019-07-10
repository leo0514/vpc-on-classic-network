---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-07"

keywords: vpc, address prefix, region, subnet, zone, reserved, IP, ranges, deleting, creating, CIDR

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# IP アドレス範囲、アドレス接頭部、リージョン、およびサブネットの理解
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (リンクされたヘルプ・トピック)

この資料では、{{site.data.keyword.cloud}} VPC のリージョン、アドレス接頭部、サブネットの関係について説明します。

* VPC は、リージョンにデプロイされ、バインドされます。
* その 1 つのリージョン内で、VPC は複数のゾーンにまたがることができます。
* アドレス接頭部によって、1 つの VPC がまたがっている複数のゾーン間の通信が可能になります。各 VPC には、またがっているゾーンごとに、ショートカットのデフォルトのアドレス接頭部が与えられます。
* 1 つのアドレス接頭部の範囲内に、複数のサブネットが作成されます。つまり、サブネットは、単一の既存のアドレス接頭部の一部である必要があります。
* サブネットに対して [RFC 1918](https://tools.ietf.org/html/rfc1918) で定義された範囲 (`10.0.0.0/8`、`172.16.0.0/12`、または `192.168.0.0/16`) 以外の IP 範囲を使用すると、そのサブネットに接続されたインスタンスがパブリック・インターネットの一部に到達できなくなる可能性があります。

## IBM Cloud VPC とリージョン
{: #ibm-cloud-vpc-and-regions}

{{site.data.keyword.cloud}} VPC は 1 つのリージョンにデプロイされますが、複数のゾーンをまたぐことができます。各リージョンには複数のゾーンが含まれています。ゾーンは、独立したフォールト・ドメインを表します。

## IBM Cloud VPC とアドレス接頭部
{: #ibm-cloud-vpc-and-address-prefixes}

アドレス接頭部によって、別々のゾーンにある {{site.data.keyword.cloud_notm}} VPC インスタンスの間の通信が可能になります。これらは、宛先インスタンスが配置されているゾーンにデータを送信するために_暗黙ルーター_が必要とするルーティング情報を提供します。各サブネットは、アドレス接頭部の一部である必要があります。{{site.data.keyword.cloud_notm}} VPC は、「ローカル」サブネットや「到達不能」サブネットの概念をサポートしていません。

_暗黙ルーター_は、1 つの VPC 内に作成されたすべてのサブネットをつなぐ内在的なネットワーク接続です。
{: note}

各 {{site.data.keyword.cloud_notm}} VPC では、ゾーンごとに最大 5 つのアドレス接頭部を利用できます。すぐに始められるように、{{site.data.keyword.cloud_notm}} VPC では各ゾーンのデフォルトのアドレス接頭部が定義されています (以下の表を参照)。ただし、ベスト・プラクティスとして、デプロイする前に VPC アドレス指定計画を設計することをお勧めします。

### VPC デフォルト・アドレス接頭部
{: #default-vpc-address-prefixes}

新しい VPC が作成されると、リージョンとゾーンに基づいてデフォルトのアドレス接頭部が以下のように割り当てられます。

[クラシック・アクセス
VPC] (/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) のデフォルト・アドレス接頭部はこれとは異なります。
{: important}

ゾーン         | アドレス接頭部 
---------------|---------------
`us-south-1`   | `10.240.0.0/18`
`us-south-2`   | `10.240.64.0/18`
`us-south-3`   | `10.240.128.0/18`
`eu-de-1`      | `10.243.0.0/18`
`eu-de-2`      | `10.243.64.0/18`
`eu-de-3`      | `10.243.128.0/18`
`jp-tok-1`     | `10.244.0.0/18`
`jp-tok-2`     | `10.244.64.0/18`
`jp-tok-3`     | `10.244.128.0/18`

新しいゾーンまたはリージョンには別のデフォルト接頭部が割り当てられる予定です。

### アドレス接頭部と IBM Cloud コンソール UI
{: #address-prefixes-and-the-ibm-cloud-console-ui}

IBM Cloud コンソール UI を使用して VPC を作成するときには、アドレス接頭部がシステムで自動的に選択されるので、ユーザーはそのデフォルト接頭部の中でサブネットを作成する必要があります。 このアドレス体系がお客様の要件に適していない場合は、VPC の作成後にアドレス接頭部をカスタマイズできます。 その後、カスタマイズしたアドレス接頭部にサブネットを作成し、デフォルト接頭部を使用して作成したサブネットを削除できます。

IBM Cloud コンソール UI で BYOIP を使用するためには、この回避策が必要です。
{:note}

## IBM Cloud VPC とサブネット
{: #ibm-cloud-vpc-and-subnets}

{{site.data.keyword.cloud_notm}} VPC はサブネットに分割できます。1 つの {{site.data.keyword.cloud_notm}} VPC 内のすべてのサブネットは、暗黙ルーターによるプライベート L3 ルーティングによって相互にアクセスできます。ルーターや経路をセットアップする必要はありません。

VPC のサブネットについて簡単に説明します。

* サブネットには、各自が指定する IP アドレス範囲があります。
* サブネットは 1 つのゾーンにバインドされ、複数のゾーンまたはリージョンをまたぐことはできません。
* {{site.data.keyword.cloud_notm}} VPC のゾーン全体を 1 つのサブネットにすることができます。
* VPC 内にサブネットを作成する前に、VPC を作成する必要があります。
* IPv6 には対応していません。
* VSI をサブネットに関連付けたり、関連付けを解除したりできます (この場合、vNIC を追加して帯域幅を選択する必要があります)。
* 各サブネットは、そのサブネットをバインドするゾーンに属するアドレス接頭部の一部である必要があります。

自分の所有するパブリック IPv4 アドレス範囲を {{site.data.keyword.cloud_notm}} VPC アカウントに持ち込む (BYOIP) ことができます。BYOIP を使用する場合、{{site.data.keyword.cloud_notm}} は、提供されたアドレスとの間でパケットを送信する {{site.data.keyword.cloud_notm}} リソース上にそれらの IPv4 アドレスを構成する必要があります。そのため、提供された IPv4 範囲を {{site.data.keyword.cloud_notm}} で使用すると、このサービスの使用を通して、それらの IP アドレスが IBM のサポート・スタッフおよび第三者に公開される可能性があります。
{:important}

![IBM Cloud VPC の概要](images/vpc-experience.svg "IBM Cloud VPC の概要")

### サブネットのアドレス接頭部の使用
{: #using-address-prefixes-for-subnets}

各サブネットは1 つのアドレス接頭部内になければなりません。
 * 新しいサブネットの場合、既存のアドレス接頭部から IP アドレス範囲を選択できます。
 * ゾーンのアドレス接頭部が適切でない場合は、(API、CLI、または UI を使用して) 既存のアドレス接頭部の 1 つを編集するか、ゾーンに新しいアドレス接頭部を追加することができます。

### 使用可能な IP アドレス
{: #available-ip-addresses}

**RFC 1918** で定義されている使用可能な IP アドレスを以下に示します。

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

許容範囲 (前のセクションで説明) 以外の IP 範囲をサブネットに使用すると、そのサブネットに接続されたインスタンスがパブリック・インターネットの一部に到達できなくなる可能性があります。

### サブネット作成方法の詳細
{: #more-about-creating-a-subnet}

VPC のサブネットは、以下の 2 つの方法で指定できます。
  * サブネットを作成するには、サポートされるアドレスの数 (例: 1024) など、必要なサブネットのサイズを指定します
  * CIDR 範囲 (10.0.0.8/29 など) を指定してサブネットを作成できます

CIDR を使用して 1024 ブロックを指定する方法の例として、サブネット・サイズではなく CIDR 範囲を指定する場合、IPv4 ブロック `192.168.100.0/ 22` は、`192.168.100.0` から `192.168.103.255` までの 1024 個の IPv4 アドレスを表します。
{:tip}

