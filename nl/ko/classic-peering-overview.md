---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: peering, classic, infrastructure, VRF, resources

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

# VPC용 클래식 피어링
{: #setting-up-classic-peering-for-vpc}

{{site.data.keyword.cloud}} VPC는 클래식 IBM Cloud 리소스와 피어링할 수 있습니다. 피어링할 수 있도록 계정이 몇 가지 요구사항을 충족해야 합니다.

클래식 피어링이 사용으로 설정된 VPC를 작성하는 방법에 대한 단계별 지시사항을 보려면 이 프로시저를 자세히 설명하는 [VPC 인프라 문서](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc)를 참조하십시오.

## 요구사항
{: #classic-peering-requirements}

1. 작성 시 VPC를 "클래식 피어링"용으로 지정해야 하며, 나중에 지정할 수 없습니다.

2. VPC를 피어링 중인 연결된 계정이 VRF 사용 계정이어야 합니다. VRF에 대한 자세한 정보는 [이 설명 문서](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud)를 참조하십시오.

3. 또한 클래식 인프라 측에서 사용 중인 각 인스턴스의 클래식 사용 VPC에 라우트를 다시 추가해야 합니다.

## 제한사항
{: #classic-peering-restrictions}

클래식 피어링의 경우 지역당 하나의 VPC만 사용으로 설정할 수 있습니다.
