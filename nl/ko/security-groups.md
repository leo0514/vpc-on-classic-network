---

copyright:
  years: 2019

lastupdated: "2019-05-20"

keywords: security groups, traffic, firewall, stateful, filtering

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
{:DomainName: data-hd-keyref="DomainName"}

# 보안 그룹 사용
{: #using-security-groups}
[comment]: # (링크된 도움말 항목)

보안 그룹을 사용하면 IP 주소를 기반으로 하여 가상 서버 인스턴스(VSI)의 각 네트워크 인터페이스에 대한 필터링을 설정하는 규칙을 편리하게 적용할 수 있습니다. 새 보안 그룹 리소스를 작성할 경우 원하는 네트워크 트래픽 패턴을 작성하도록 이를 업데이트합니다.

보안 그룹과 ACL 간의 특성 비교는 [비교 표](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists)를 참조하십시오.

기본적으로 보안 그룹은 모든 트래픽을 거부합니다. 규칙이 보안 그룹에 추가되면서 보안 그룹이 허용하는 트래픽을 정의합니다.

규칙은 _stateful_입니다. 즉, 허용된 트래픽에 대응하는 역 트래픽이 자동으로 허용됩니다. 예를 들어, 포트 80에서 인바운드 TCP 트래픽을 허용하는 규칙은 추가 규칙 없이도 포트 80에서 원래 호스트로 응답하는 아웃바운드 TCP 트래픽도 허용합니다.

보안 그룹의 범위는 단일 VPC입니다. 이 범위는 보안 그룹이 동일한 VPC 내의 VSI의 네트워크 인터페이스에 _한정하여_ 연결될 수 있음을 의미합니다.

VSI가 보안 그룹이 지정되지 않고 작성된 경우, VSI의 기본 네트워크 인터페이스가 해당 VSI의 VPC의 _기본_ 보안 그룹에 배치됩니다. 이 {{site.data.keyword.cloud}} VPC 릴리스에는 특정 트래픽을 허용하는 기본 보안 그룹이 정의되어 있습니다. 자세한 정보는 [기본 보안 그룹 업데이트](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-updating-the-default-security-group)를 참조하십시오.

REST API, CLI 또는 UI를 사용하여 보안 그룹을 설정할 수 있습니다.

* [API를 사용하여 보안 그룹 설정](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-apis)
* [CLI를 사용하여 보안 그룹 설정 ](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [UI를 사용하여 보안 그룹 설정](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
