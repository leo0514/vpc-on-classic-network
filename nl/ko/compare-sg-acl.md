---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# 보안 그룹 및 액세스 제어 목록 비교
{: #compare-security-groups-and-access-control-lists}

보안 그룹 및 액세스 제어 목록은 사용자가 지정하는 규칙을 사용하여 {{site.data.keyword.cloud}} VPC의 서브넷 및 인스턴스 간의 트래픽을 제어하는 방법을 제공합니다.

다음은 보안 그룹 및 ACL 간의 몇 가지 핵심 차이점을 요약한 표입니다.

|  |       보안 그룹 |   ACL    |
|-------------|-----------------|---------|
| 제어 레벨  | VSI 인스턴스    | 서브넷  |
| 상태 |Stateful - 인바운드 연결이 허용되면 응답할 수 있음 |Stateless - 인바운드 및 아웃바운드 연결이 모두 명시적으로 허용되어야 함 |
| 지원되는 규칙 |허용 규칙만 사용 |허용 및 거부 규칙 사용|
| 규칙이 적용되는 방법 |모든 규칙이 고려됨 | 규칙이 순서대로 처리됨 |
| 연관된 리소스에 대한 관계 | 인스턴스가 다중 보안 그룹과 연관될 수 있음| 다중 서브넷이 동일한 ACL과 연관될 수 있음|
