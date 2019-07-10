---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: IPv4, ranges, subnets, CIDR, 1918

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


# VPC の IP 範囲の選択
{: #choosing-ip-ranges-for-your-vpc}

次のような CIDR 表記を使用します。

* `<IPv4 address>/number` (VPC アドレスの例: 10.10.0.0/16)。

IPv4 の最後の 16 ビット (アドレス 65,536 個分) を 0 として予約しておけば、同じ {{site.data.keyword.cloud}} VPC 内のさまざまなサブネット IP アドレスに使用できます (サブネット IP アドレスの例: 10.10.1.0/24)。

CIDR 表記は、[RFC 1518](https://tools.ietf.org/html/rfc1518) および [RFC 1519](https://tools.ietf.org/html/rfc1519) で規定されています。
{: note}

サブネットに対して [RFC 1918](https://tools.ietf.org/html/rfc1918) で定義された範囲 (`10.0.0.0/8`、`172.16.0.0/12`、または `192.168.0.0/16`) 以外の IP 範囲を使用すると、そのサブネットに接続されたインスタンスがパブリック・インターネットの一部に到達できなくなる可能性があります。

CIDR 表記を初めて使用するユーザーへの注記として、スラッシュの後の数字が小さいほど、割り振る IP アドレスの数が**増える**点に注意してください。これは、スラッシュの後の数字がサブネットのプレフィックス・マスクの先行 1 ビットの数を表すためです。

以下の表に、サブネットの使用可能なアドレス数を CIDR ブロック・サイズの指定別に示します。

| CIDR ブロック・サイズ | 使用可能なアドレス数 |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

詳細情報が必要な場合は、_クラスレス・ドメイン間ルーティング_ (CIDR) に関する多数の優れた記事をオンラインで見つけることができます。
