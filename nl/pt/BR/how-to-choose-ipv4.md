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


# Escolhendo intervalos de IP para sua VPC
{: #choosing-ip-ranges-for-your-vpc}

Use a notação CIDR, tal como:

* `<IPv4 address>/number` (exemplo de endereço do VPC: 10.10.0.0/16).

Você desejará reservar os últimos 16 bits (65.536 endereços) do IPv4 como 0s para que seja possível usá-los para vários endereços IP de sub-rede dentro da mesma VPC do {{site.data.keyword.cloud}} (exemplo de endereço IP de sub-rede: 10.10.1.0/24).

A notação CIDR é definida no [RFC 1518](https://tools.ietf.org/html/rfc1518) e no [RFC 1519](https://tools.ietf.org/html/rfc1519).
{: note}

Se você usar um intervalo de IPs fora dos intervalos definidos pelo [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` ou `192.168.0.0/16`) para uma sub-rede, as instâncias anexadas a essa sub-rede poderão ser incapazes de acessar partes da Internet pública.

Lembre-se de que, caso você seja novo na notação CIDR, quanto menor o número após a barra, **mais** endereços IP você estará alocando, pois o número após a barra representa o número de bits 1 iniciais na máscara de prefixo da sub-rede.

A tabela a seguir lista o número de endereços disponíveis em uma sub-rede com base em seu tamanho de bloco CIDR especificado:

| Tamanho do bloco CIDR | Endereços disponíveis |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Se você precisa de mais informações, vários artigos excelentes referentes a _Classless Inter-Domain Routing_ (CIDR) podem ser localizados on-line.
