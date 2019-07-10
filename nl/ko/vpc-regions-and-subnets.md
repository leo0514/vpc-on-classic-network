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

# IP 주소 범위, 주소 접두부, 지역 및 서브넷 이해
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (링크된 도움말 항목)

이 문서에서는 {{site.data.keyword.cloud}} VPC에 대한 지역, 주소 접두부 및 서브넷 간의 관계에 대해 설명합니다.

* VPC는 배치되어 한 지역에 바인드됩니다.
* 해당 한 지역 내에서 VPC는 다중 구역에 걸쳐 있을 수 있습니다.
* 주소 접두부를 사용하여 VPC가 걸쳐 있는 구역 간에 통신할 수 있습니다. 각 VPC는 걸쳐 있는 각 구역에 대한 단축 기본 주소 접두부를 얻습니다.
* 서브넷은 주소 접두부의 범위 내에 작성되며, 이는 서브넷이 하나의 기존 주소 접두부 내에 완전히 포함되어야 함을 의미합니다.
* 서브넷에 대한 [RFC 1918](https://tools.ietf.org/html/rfc1918)(`10.0.0.0/8`, `172.16.0.0/12` 또는 `192.168.0.0/16`)에서 정의된 범위를 벗어난 IP 범위를 사용하는 경우 해당 서브넷에 연결된 인스턴스가 공용 인터넷의 일부에 연결할 수 없습니다.

## IBM Cloud VPC 및 지역
{: #ibm-cloud-vpc-and-regions}

{{site.data.keyword.cloud}} VPC는 지역에 배치되지만 다중 구역을 포함할 수 있습니다. 각 지역은 다중 구역을 포함하며 구역은 독립적인 결함 도메인을 나타냅니다.

## IBM Cloud VPC 및 주소 접두부
{: #ibm-cloud-vpc-and-address-prefixes}

주소 접두부를 사용하면 여러 구역의 {{site.data.keyword.cloud_notm}} VPC 인스턴스 간에 통신할 수 있습니다. 대상 인스턴스가 있는 구역에 데이터를 전송하기 위해 _내재적 라우터_에 필요한 라우팅 정보를 제공합니다. 각 서브넷은 주소 접두부에 포함되어야 합니다. {{site.data.keyword.cloud_notm}} VPC는 로컬" 또는 "도달 불가능" 서브넷의 개념을 지원하지 않습니다.

_내재적 라우터_는 VPC 내에 작성된 모든 서브넷 간의 내재적 네트워크 연결입니다.
{: note}

각 {{site.data.keyword.cloud_notm}} VPC에는 각 구역마다 최대 5개의 주소 접두부가 있을 수 있습니다. 시작하는 데 도움을 주기 위해 {{site.data.keyword.cloud_notm}} VPC가 각 구역에 대한 기본 주소 접두부(다음 표 참조)를 정의하지만 우수 사례로 VPC 주소 지정 플랜을 배치하기 전에 먼저 디자인해야 합니다. 

### VPC 기본 주소 접두부
{: #default-vpc-address-prefixes}

새 VPC가 작성될 때 다음과 같이 지역 및 구역을 기반으로 하여 기본 주소 접두부가 지정됩니다.

[클래식 액세스 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes)에는 다양한 기본 주소 접두부 세트가 있습니다.
{: important}

구역           |주소 접두부 
---------------|---------------
`us-south-1`   |`10.240.0.0/18`
`us-south-2`   |`10.240.64.0/18`
`us-south-3`   |`10.240.128.0/18`
`eu-de-1`      |`10.243.0.0/18`
`eu-de-2`      |`10.243.64.0/18`
`eu-de-3`      |`10.243.128.0/18`
`jp-tok-1`     |`10.244.0.0/18`
`jp-tok-2`     |`10.244.64.0/18`
`jp-tok-3`     |`10.244.128.0/18`

다양한 기본 접두부가 새 구역 또는 지역에 지정됩니다.

### 주소 접두부 및 IBM Cloud 콘솔 UI
{: #address-prefixes-and-the-ibm-cloud-console-ui}

IBM Cloud 콘솔 UI를 사용하여 VPC를 작성하는 경우, 시스템이 자동으로 주소 접두부를 선택하고 사용자로 하여금 해당 기본 접두부 내에서 서브넷을 작성하도록 요구합니다. 주소 스킴이 요구사항에 적합하지 않으면 VPC를 작성한 후에 주소 접두부를 사용자 정의할 수 있습니다. 그런 다음 사용자 정의된 주소 접두부 내에서 서브넷을 작성하고 기본 접두부를 사용하여 작성한 서브넷을 삭제할 수 있습니다.

IBM Cloud 콘솔 UI를 통해 BYOIP를 사용해야 하는 경우에 이 임시 해결책이 필요합니다.
{:note}

## IBM Cloud VPC 및 서브넷
{: #ibm-cloud-vpc-and-subnets}

{{site.data.keyword.cloud_notm}} VPC를 서브넷으로 나눌 수 있습니다. {{site.data.keyword.cloud_notm}} VPC 내의 모든 서브넷은 내재적 라우터를 사용하여 사설 L3 라우팅을 통해 서로 연결될 수 있습니다. 사용자가 라우터를 설정할 필요가 없습니다.

다음은 VPC 내의 서브넷에 대한 유용한 정보입니다.

* 서브넷은 사용자가 지정하는 IP 주소 범위로 구성됩니다.
* 서브넷은 단일 구역에 바인드되며 다중 구역 또는 지역에 걸쳐 있을 수 없습니다.
* 서브넷은 {{site.data.keyword.cloud_notm}} VPC 내의 전체 구역에 걸쳐 있을 수 있습니다.
* 해당 VPC 내에 서브넷을 작성하기 전에 VPC를 작성해야 합니다.
* IPv6 지원은 사용할 수 없습니다.
* VSI를 서브넷에 연결하거나 연결 취소할 수 있습니다. (vNIC를 추가하고 대역폭을 선택해야 합니다.)
* 각 서브넷은 해당 서브넷이 바인드된 구역에 속한 주소 접두부 내에 포함되어야 합니다.

사용자 자신의 공인 IPv4 주소 범위(BYOIP)를 사용자의 {{site.data.keyword.cloud_notm}} VPC 계정으로 가져올 수 있습니다. BYOIP를 사용하는 경우, {{site.data.keyword.cloud_notm}}가 {{site.data.keyword.cloud_notm}} 리소스에서 해당 IPv4 주소를 구성해야 합니다. 그러면 제공된 주소에 대해 패킷이 전송됩니다. 따라서 {{site.data.keyword.cloud_notm}}에서 제공된 IPv4 범위를 사용하면 해당 IP 주소가 서비스 사용의 일부로 IBM 지원 센터의 담당자 및 서드파티에 공개됩니다.
{:important}

![IBM Cloud VPC 개요](images/vpc-experience.svg "IBM Cloud VPC 개요"){: caption="Figure: A graphical representation of a VPC showing zones, regions, and subnets." caption-side="top"}

### 서브넷에 대해 주소 접두부 사용
{: #using-address-prefixes-for-subnets}

각 서브넷은 주소 접두부 내에 있어야 합니다.
 * 새 서브넷의 경우, 기존 주소 접두부에서 IP 주소 범위를 선택할 수 있습니다.
 * 구역의 주소 접두부가 적합하지 않으면 API, CLI 또는 UI를 사용하여 구역에 대해 기존 주소 접두부 중 하나를 편집하거나 새 주소 접두부를 추가할 수 있습니다.

### 사용 가능한 IP 주소
{: #available-ip-addresses}

다음은 **RFC 1918**에 정의되어 있는 사용 가능한 IP 주소입니다.

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

허용 가능한 서브넷 범위(이전 섹션에서 제공됨)를 벗어난 IP 범위를 사용하는 경우 해당 서브넷에 연결된 인스턴스가 공용 인터넷의 일부에 연결할 수 없습니다.

### 서브넷 작성 정보
{: #more-about-creating-a-subnet}

다음과 같은 두 가지 방법으로 VPC에 대한 서브넷을 지정할 수 있습니다.
  * 지원되는 주소의 수(예: 1024) 등의 필요한 서브넷의 크기를 제공하여 서브넷을 작성할 수 있습니다.
  * CIDR 범위(예: 10.0.0.8/29)를 제공하여 서브넷을 작성할 수 있습니다.

CIDR을 사용하여 1024 블록을 지정하는 방법의 예를 들면, 서브넷 크기 대신 CIDR 범위를 지정하는 경우, IPv4 블록 `192.168.100.0/22`는 `192.168.100.0`부터 `192.168.103.255`까지의 1024 IPv4 주소를 나타냅니다.
{:tip}

