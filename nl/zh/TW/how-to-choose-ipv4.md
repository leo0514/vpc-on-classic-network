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


# 選擇 VPC 的 IP 範圍
{: #choosing-ip-ranges-for-your-vpc}

使用 CIDR 表示法，例如：

* `<IPv4 address>/number`（VPC 位址範例：10.10.0.0/16）。

您想要將 IPv4 的最後 16 個位元（65,536 位址）保留為 0，這樣您就可以在相同的 {{site.data.keyword.cloud}} VPC（子網路 IP 位址範例：10.10.1.0/24）內，將它們用於各種子網路 IP 位址。

CIDR 表示法以 [RFC 1518](https://tools.ietf.org/html/rfc1518) 及 [RFC 1519](https://tools.ietf.org/html/rfc1519) 進行定義。
{: note}

如果您針對子網路使用 [RFC 1918](https://tools.ietf.org/html/rfc1918) 所定義範圍之外的 IP 範圍（`10.0.0.0/8`、`172.16.0.0/12` 或 `192.168.0.0/16`），則連接至該子網路的實例可能無法連接公用網際網路的某些部分。

萬一您是第一次使用 CIDR 表示法，請記住，斜線之後的數字越小，您配置的 IP 位址**越多**，因為斜線之後的數字代表子網路的字首遮罩前面的 1 位元數目。

下表根據指定的 CIDR 區塊大小，列出子網路中的可用位址數目：

| CIDR 區塊大小 | 可用的位址 |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

如果您需要更多資訊，可以在線上找到關於_無類別內部網域遞送_ (CIDR) 的許多優秀文章。
