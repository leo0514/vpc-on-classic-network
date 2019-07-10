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


# VPC용 IP 범위 선택
{: #choosing-ip-ranges-for-your-vpc}

다음과 같은 CIDR 표기법을 사용하십시오.

* `<IPv4 address>/number`(VPC 주소 예: 10.10.0.0/16)

동일한 {{site.data.keyword.cloud}} VPC 내의 다양한 서브넷 IP 주소용으로 사용할 수 있도록 IPv4의 마지막 16비트(65,536개의 주소)를 0으로 예약할 수 있습니다(서브넷 IP 주소 예: 10.10.1.0/24).

CIDR 표기법은 [RFC 1518](https://tools.ietf.org/html/rfc1518) 및 [RFC 1519](https://tools.ietf.org/html/rfc1519)에 정의됩니다.
{: note}

서브넷에 대한 [RFC 1918](https://tools.ietf.org/html/rfc1918)(`10.0.0.0/8`, `172.16.0.0/12` 또는 `192.168.0.0/16`)에서 정의된 범위를 벗어난 IP 범위를 사용하는 경우 해당 서브넷에 연결된 인스턴스가 공용 인터넷의 일부에 연결할 수 없습니다.

CIDR 표기법을 처음 사용하는 경우, 슬래시 뒤의 숫자가 적을수록 **더 많은** IP 주소를 할당함에 유의하십시오. 슬래시 뒤의 숫자는 서브넷의 접두부 마스크 내의 선행 1의 수를 나타내기 때문입니다.

다음 표에는 지정된 CIDR 블록 크기를 기반으로 서브넷에서 사용 가능한 주소의 수가 나열되어 있습니다.

| CIDR 블록 크기 | 사용 가능한 주소 |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

자세한 정보가 필요하면 온라인에서 찾을 수 있는 _CIDR(Classless Inter-Domain Routing)_ 관련 여러 우수 기사를 참조하십시오.
