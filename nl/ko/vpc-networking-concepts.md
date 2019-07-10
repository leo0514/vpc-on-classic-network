---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: VRF, router, hypervisor, address prefixes, classic access, implicit router, packet flows, NAT, data flows

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

# 비밀 VPC
{: #vpc-behind-the-curtain}

이 페이지는 VPC 네트워킹과 관련하여 배후에서 발생하는 자세한 개념도를 표시합니다. 독자는 네트워킹 배경에 대해 어느 정도 알고 있어야 합니다.

## 네트워크 격리
{: #network-isolation}

VPC 네트워크 격리는 다음 세 개의 레벨에서 수행됩니다.

* **하이퍼바이저**: VSI(가상 서버 인스턴스)는 하이퍼바이저 자체에 의해 격리됩니다. 동일한 VPC에 있지 않은 경우 동일한 하이퍼바이저에 의해 호스트된 다른 VSI에 VSI를 직접 연결할 수 없습니다.

* **네트워크**: 격리는 **가상 사설 클라우드**(VNI)의 사용을 통해 네트워크 레벨에서 발생합니다. 이러한 ID의 범위는 로컬 구역입니다. 이러한 VNI는 VPC의 구역에 들어가는 모든 데이터 패킷에 추가됩니다. VSI에서 전송된 경우 하이퍼바이저에서 들어가거나 내재적 라우팅 기능에서 전송된 경우 클라우드에서 구역에 들어갑니다.

구역에서 나가는 패킷에서는 VNI가 연결 해제됩니다. 패킷이 내재적 라우팅 기능을 통해 들어가서 대상 구역에 도달하면 내재적 라우터가 항상 해당 구역에 대한 적절한 VNI를 추가합니다.
{: note}

* **라우터**: _내재적 라우터 기능_은 클라우드 백본에서 MPLS(다중 프로토콜 레이블 전환)와 함께 VPN 및 **가상 라우팅 기능**(VRF)을 제공하여 각 VPC에 대한 격리를 제공합니다. 각 VPC의 VRF는 고유 ID를 가지며, 이 격리를 통해 각 VPC는 IPv4 주소 공간의 자체 사본에 대한 액세스를 가집니다. MPLS VPN은 클라우드의 모든 에지 연합을 허용합니다(클래식 인프라, 직접 링크, VPC).

## 주소 접두부
{: #address-prefixes}

주소 접두부는 대상 VSI가 위치해 있는 가용성 구역과 관계없이 _대상 VSI_를 찾기 위해 VPC의 내재적 라우팅 기능에서 사용된 요약 정보입니다. 주소 접두부의 기본 기능은 라우팅 오류를 피하는 한편 MPLS VPN을 통한 라우팅을 최적화하는 것입니다. VC에서 작성된 모든 서브넷에는 VPC의 모든 다른 VSI에서 VPC의 모든 VSI에 연결할 수 있도록 주소 접두부에 포함되어야 합니다.

## 데이터 패킷 플로우 및 내재적 라우터
{: #data-packet-flows-and-the-implicit-router}

6개 유형의 VSI 데이터 패킷 플로우가 VPC에서 발생합니다. 복잡도 오름차순에 따라 플로우는 다음과 같습니다.

* Intra-subnet, intra-host(동일한 하이퍼바이저)
* Intra-subnet, inter-host
* Inter-subnet, intra-zone
* Inter-subnet, inter-zone
* Extra-VPC service(IaaS 또는 CSE 액세스용)
* Extra-VPC Internet(인터넷 액세스용)

**Intra-subnet, intra-host** 데이터 플로우: 가장 단순합니다. 패킷은 하이퍼바이저의 VSI 간을 흐르며, 하이퍼바이저를 떠나지 않습니다.

**Intra-subnet, inter-host** 데이터 플로우: 이러한 플로우에는 하이퍼바이저에서 나가는 패킷이 포함됩니다. 각 패킷은 데이터 격리를 보장하기 위해 적절한 VNI(가상 네트워크 ID)로 태그 지정된 후 대상 VSI를 호스트하는 대상 하이퍼바이저로 전송됩니다. 대상 하이퍼바이저는 VNI를 연결 해제하고 대상 VSI에 데이터 패킷을 전달합니다.

**Inter-subnet, intra-zone** 데이터 플로우: 이러한 플로우에는 VPC의 내재적 라우터 기능을 활용하는 패킷이 포함되며, 이는 VPC에서 작성된 모든 서브넷을 연결합니다. 이는 데이터 페킷을 올바른 대체 하이퍼바이저로 라우트합니다. 대상 하이퍼바이저가 소스 하이퍼바이저와 다른 경우, 대상 패킷이 적절한 VNI로 태그 지정되고 대상 하이퍼바이저로 전송됩니다. 그런 다음 VNI가 연결 해제되고 데이터 패킷이 대상 VSI로 전달됩니다. (마지막 단계는 이전 유형의 데이터 플로우에 설명된 대로 동일합니다.)

**Inter-subnet, inter-zone** 데이터 플로우: 이러한 플로우의 경우 내재적 라우터 기능이 VNI를 제거하고 클라우드 백본에서 전송하기 위해 VPC의 MPLS VPN에서 패킷을 전달합니다. 대상 구역에서는 내재적 라우터 기능이 적절한 VNI로 데이터 패킷의 태그를 지정합니다. 그러면 패킷이 대상 하이퍼바이저로 전달되며, 여기서는 앞서 설명한 대로 데이터 패킷이 대상 VSI로 전달될 수 있도록 VNI가 다시 연결 해제됩니다.

**Extra-vpc service** 데이터 플로우: IaaS 또는 IBM Cloud 서비스 엔드포인트(CSE) 서비스로 향하는 패킷은 VPC의 내재적 라우터 기능 및 네트워크 주소 변환(NAT) 기능을 활용합니다. 변환 기능은 VSI 주소를 IPv4 주소로 대체하며, 이를 통해 요청 중인 IaaS 또는 CSE 서비스에 대해 VPC를 식별합니다.

**Extra-vpc Internet** 데이터 플로우: 인터넷을 향한 패킷은 가장 복잡합니다. VPC의 내재적 라우터 기능을 활용하는 외에도, 이러한 플로우 각각은 내재적 라우터의 두 네트워크 주소 변환(NAT) 기능 중 하나에 의존합니다.

  * 연결된 모든 서브넷을 제공하는 공용 게이트웨이 기능을 통한 명시적 일대다 NAT.
  * 개별 VSI에 지정된 일대일 NAT.

NAT 변환 후에 내재적 라우터는 클라우드 백본을 사용하여 이러한 인터넷 대상 패킷을 인터넷에 전달합니다.

## 클래식 액세스
{: #classic-access}

VPC용 [**클래식 액세스**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc) 기능은 {{site.data.keyword.cloud}} 클래식 인프라 계정에서 VRF ID를 다시 사용하여 설정됩니다. 이 구현은 클래식 인프라 계정에서 사용되는 동일한 MPLS VPN을 결합하기 위해 VPC의 내재적 라우터 기능을 허용합니다. 따라서 VPC는 기존 직접 링크 연결을 통해 연결할 수 있는 클래식 리소스 및 기타에 대한 액세스를 가집니다.
