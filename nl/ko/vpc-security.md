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

# IBM Cloud VPC의 보안
{: #security-in-your-ibm-cloud-vpc}

보안 그룹(SG)을 사용하여 네트워크 트래픽을 제어하거나, 네트워크 액세스 제어 목록(ACL)을 사용하거나, 두 가지 유형의 제어를 모두 사용하여 VPC 및 워크로드를 안전하게 보호할 수 있습니다. 주의:

* 보안 그룹은 인스턴스별(VSI)로 트래픽을 제어합니다.
* 액세스 제어 목록은 서브넷별로 트래픽을 제어합니다.

## 보안 개요
{: #security-overview}

* 서브넷에 대한 트래픽은 액세스 제어 목록(ACL)에 의해 제어될 수 있습니다.
* 보안 그룹(SG)은 VSI 레벨에서 트래픽을 제어할 수 있습니다.
* ACL에서 안내하는 인터넷에 대한 서브넷 액세스를 위해 공용 게이트웨이를 설정하십시오.
* SG에서 안내하는 인터넷에 대한 VSI 액세스를 위해 유동 IP를 설정하십시오.

![IBM VPC 연결 및 보안](images/vpc-connectivity-and-security.svg "IBM VPC 연결 및 보안"){: caption="Figure: Security groups and ACLs add security to your subnets and instances." caption-side="top"}

## 정의
{: #definitions}

이 [용어집](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary)은 ACL과 SG의 정의 및 설명, 사용자가 이를 사용하여 수행할 수 있는 조치를 제공합니다. 다음 섹션에서는 ACL과 보안 그룹의 기본 기능 및 VPC가 엔드-투-엔드 암호화를 지원하는 방법에 대해 설명합니다.

### 액세스 제어 목록(ACL, Access Control List)
{: #access-control-list}

**액세스 제어 목록(ACL)**은 서브넷에 대한 인바운드 및 아웃바운드 트래픽을 관리(즉, 허용 또는 거부)할 수 있습니다. ACL은 Stateless이며 이는 인바운드 및 아웃바운드 규칙이 별도로 명시적으로 지정되어야 함을 의미합니다. 각 ACL은 *소스 IP*, *소스 포트*, *대상 IP*, *대상 포트* 및 *프로토콜*을 기반으로 하는 규칙으로 구성됩니다.

{{site.data.keyword.cloud}} VPC에서 모든 서브넷은 기본 ACL을 사용하여 작성됩니다. 즉, 인바운드 및 아웃바운드 트래픽을 허용하나 고객이 사용자 정의 ACL을 작성할 수 있습니다. 항상 하나의 ACL만 서브넷에 연결되나 하나의 ACL이 다중 서브넷에 연결될 수 있습니다. ACL 사용 방법에 대한 자세한 정보는 [ACL 안내서](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)를 참조하십시오.

### 보안 그룹(SG, Security Group)
{: #security-group}

**보안 그룹**은 하나 이상의 서버(VSI)에 대한 트래픽을 제어하는 가상 방화벽 역할을 수행합니다. 보안 그룹은 연관된 VSI에 대한 트래픽 허용 여부를 지정하는 규칙 콜렉션입니다.

고객이 VSI를 작성할 때 하나 이상의 보안 그룹을 해당 VSI와 연관시킬 수 있습니다. 올바른 권한이 제공되면 고객이 IBM 콘솔, CLI 또는 API를 사용하여 보안 그룹 규칙을 수정할 수 있습니다.

보안 그룹을 사용하는 VSI 작성 방법 및 보안 그룹의 기능에 대한 자세한 정보는 [보안 그룹 안내서](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups)를 참조하십시오.

### 엔드-투-엔드 암호화(End-to-end encryption)
{: #end-to-end-encryption}

IBM Cloud VPC는 엔드-투-엔드 암호화를 제공하지 않지만 이를 허용합니다. 예를 들어, 가상 서버(예: 포트 443의 HTTPS 서버)에 보안 엔드포인트가 있는 경우, 유동 IP를 해당 서버에 연결한 다음 연결이 클라이언트에서 포트 443의 서버로 엔드-투-엔드 암호화됩니다.  경로 내에서 어떠한 것도 강제로 복호화되지 않습니다.

그러나, 포트 80에서 HTTP 등의 비보안 프로토콜을 사용하는 경우에는 데이터가 처음부터 끝까지 엔드-투-엔드 암호화되지 않은 평문 형식으로 있게 됩니다.
