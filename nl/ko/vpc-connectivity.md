---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: capabilities, use cases, subnets, VPN, connections, reserved, IP, IPv4, floating

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# VPC용 네트워킹 정보
{: #about-networking-for-vpc}

이 문서에서는 서브넷, 유동 IP 주소 및 VPN용 {{site.data.keyword.cloud}} VPC 연결 내의 유스 케이스 및 기능을 설명하는 개념을 찾을 수 있습니다.

![IBM VPC 연결 및 보안](images/vpc-connectivity-and-security.svg "IBM VPC 연결 및 보안"){: caption="Figure: You can subdivide a Virtual Private Cloud with subnets, and each subnet can reach the public Internet, if desired." caption-side="top"}

그림에 표시된 바와 같이,

* 서브넷은 선택적 공용 게이트웨이(PGW)를 통해 공용 인터넷에 연결할 수 있습니다.
* 인터넷으로부터 연결하도록 유동 IP 주소(FIP)를 임의의 VSI에 지정하거나 그 반대로 수행할 수 있습니다.
* IBM Cloud VPC 내의 서브넷은 사설 연결을 제공하며 사설 링크를 통해 서로 통신할 수 있습니다. 사용자가 라우터를 설정할 필요가 없습니다.
* 자세한 정보는 [VPC 인프라 정보](/docs/vpc-on-classic?topic=vpc-on-classic-about)를 참조하십시오.

## 용어
{: #terminology}

이 [용어집](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary)에는 이 IBM Cloud VPC용 문서에서 사용되는 용어에 대한 정의 및 정보가 포함되어 있습니다.

## VPC의 서브넷의 특성
{: #characteristics-of-subnets-in-the vpc}

서브넷은 지정된 IP 주소 범위(CIDR 블록)로 구성됩니다. 서브넷은 단일 구역에 바인드되고 다중 구역 또는 지역에 걸쳐 있을 수 없습니다. 그러나 서브넷은 가상 사설 클라우드 내의 전체 추상 구역에 걸쳐 있을 수 있습니다. 동일한 IBM Cloud VPC 내의 서브넷은 서로 연결됩니다.

### 구역
{: #zones}

구역은 내결함성 개선 및 대기 시간 감축을 지원하도록 디자인된 일종의 추상적 개념입니다. 구역의 특성은 다음과 같습니다.

 * 각 구역은 독립적인 결함 도메인이며 지역 내의 두 구역이 동시에 실패하는 경우는 극단적으로 드뭅니다.
 * 지역 내 구역 사이의 트래픽은 < 2ms 대기 시간을 가집니다.

### 예약된 IP 주소
{: #reserved-ip-addresses}

특정 IP 주소는 가상 사설 클라우드를 운영할 때 IBM에서 사용하기 위해 예약되어 있습니다. 다음은 예약된 주소입니다. 제공된 IP 주소는 서브넷의 CIDR 범위가 10.10.10.0/24라고 가정합니다.

  * CIDR 범위의 첫 번째 주소(10.10.10.0): 네트워크 주소
  * CIDR 범위의 두 번째 주소(10.10.10.1): 게이트웨이 주소
  * CIDR 범위의 세 번째 주소(10.10.10.2): IBM에 의해 예약됨
  * CIDR 범위의 네 번째 주소(10.10.10.3): 추후 사용을 위해 IBM에 의해 예약됨
  * CIDR 범위의 마지막 주소(10.10.10.255): 네트워크 브로드캐스트 주소

### 서브넷의 외부 연결을 위한 공용 게이트웨이 사용
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

**공용 게이트웨이(PGW)**를 사용하면 서브넷이 (서브넷에 연결된 모든 VSI와 함께) 인터넷에 연결됩니다. 서브넷은 기본적으로 개인용이나 선택적으로 PGW를 작성하고 서브넷을 PGW에 연결할 수 있습니다. 서브넷이 PGW에 연결된 후에 해당 서브넷의 모든 VSI가 인터넷에 연결될 수 있습니다.

PGW는 _다대일 NAT_를 사용합니다. 즉, 사설 주소를 가진 수천 개의 VSI가 하나의 공인 IP 주소를 사용하여 공용 인터넷에 접속합니다.PGW는 인터넷이 해당 인스턴스와의 연결을 시작할 수 있도록 허용하지 않습니다. API를 사용하여 PGW에 대해 서브넷을 연결하거나 분리하십시오.

다음은 게이트웨이 서비스의 범위를 요약한 그림입니다.

![게이트웨이 서비스](images/scope-of-gateway-services.png)

### VSI의 외부 연결을 위한 유동 IP 주소 사용
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

**유동 IP 주소**는 시스템에 의해 제공되는 IP 주소이며 공용 인터넷에서 연결할 수 있습니다.

IBM이 제공하는 사용 가능한 유동 IP 주소 풀에서 유동 IP 주소를 예약할 수 있으며 동일한 가상 사설 클라우드 내의 임의의 인스턴스를 사용하면 해당 인스턴스에 대한 vNIC를 통해 이를 연결하거나 분리할 수 있습니다. 모든 유동 IP 주소는 로드 밸런서 또는 VPN 게이트웨이 등의 다양한 유형의 가상 서버 인스턴스(VSI)와 연관될 수 있습니다.

사용자의 유동 IP 주소는 다중 인터페이스와 연관될 수 없습니다.개별 유동 IP와 연관될 VSI 상의 인터페이스를 지정해야 합니다. 해당 인터페이스에는 사설 IP 주소도 있습니다. 백엔드 시스템은 해당 인터페이스의 유동 IP 및 사설 IP 간의 _일대일 NAT_ 작업을 수행합니다. 유동 IP는 사용자가 별도로 할당 해제 조치를 요청한 경우에만 해제됩니다.

**참고:**
* **유동 IP 주소와 VSI를 연관시키면 앞에서 언급한 PGW의 다대일 NAT에서 VSI가 제거됩니다.**
* **현재, 유동 IP는 IPv4 주소만 지원합니다.**
* **자신의 공인 IP 주소를 유동 IP로 사용할 수 없습니다.**

NAT 작업에 대한 자세한 정보는 [연관된 인터넷 RFC 문서![외부 링크 아이콘](../../icons/launch-glyph.svg "외부 링크 아이콘")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}를 참조하십시오.

### 안전한 외부 연결을 위해 VPN 사용
{: #use-a-vpn-for-secure-external-connectivity}

사용자가 가상 사설망(VPN) 서비스를 사용하면 인터넷에서 IBM Cloud VPC에 안전하게 연결할 수 있습니다. 단계별 지시사항을 보려면 [IBM Console UI 안내서](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)를 참조하십시오.

**VPN 기능**
  * 데이터 전송을 처리하기 위한 작성, 읽기, 업데이트 및 삭제(CRUD) VPN 서비스(사이트 대 사이트 IPSEC VPN) 기능
  * VPN 서비스를 고객의 IBM Cloud VPC 또는 가상 네트워크와 연관시키는 기능
  * 정적 정의 또는 동적 라우팅을 사용하여 고객의 온사이트 서브넷을 식별하는 기능
  * SHA256, AES, 3DES, IKEv2 등의 보안 암호 지원
  * 기본 제공 서비스 신뢰성
  * VPN 연결 상태의 모니터링
  * 개시자 및 응답자 모드 둘 다에 대한 지원. 즉, 터널의 어느 쪽에서나 트래픽이 시작될 수 있습니다.
