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


# 为 VPC 选择 IP 范围
{: #choosing-ip-ranges-for-your-vpc}

使用 CIDR 表示法，例如：

* `<IPv4 address>/number`（VPC 地址示例：10.10.0.0/16）。

您希望将 IPv4 的最后 16 位（65,536 个地址）保留为 0，以便可以将这些地址用于同一 {{site.data.keyword.cloud}} VPC 中的各种子网 IP 地址（子网 IP 地址示例：10.10.1.0/24）。

CIDR 表示法在 [RFC 1518](https://tools.ietf.org/html/rfc1518) 和 [RFC 1519](https://tools.ietf.org/html/rfc1519) 中定义。
{: note}

如果将 [RFC 1918](https://tools.ietf.org/html/rfc1918) 定义的范围（`10.0.0.0/8`、`172.16.0.0/12` 或 `192.168.0.0/16`）之外的 IP 范围用于某个子网，那么连接到该子网的实例可能无法访问公用因特网的某些部分。

万一您不熟悉 CIDR 表示法，请记住，斜杠后的数字越小，分配的 IP 地址**越多**，因为斜杠后的数字代表子网前缀掩码中前导 1 的位数。

下表根据指定的 CIDR 块大小，列出子网中的可用地址数：

| CIDR 块大小 | 可用地址数 |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

如果您需要更多信息，可以在线找到有关_无类域间路由_ (CIDR) 的一些优秀的文章。
