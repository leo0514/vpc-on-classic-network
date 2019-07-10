---

copyright:
  years: 2018, 2019

lastupdated: "2019-05-14"

keywords: API, CLI, commands, comparison, security groups, ACL

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}


# API 및 CLI에 대한 명령 비교
{: #comparison-of-commands-for-the-api-and-ci}

이 표 세트에서는 REST API를 통해 또는 IBM Cloud CLI를 통해 호출할 수 있는 유사한 명령을 비교하는 방법을 제공합니다.

## 보안 그룹에 대한 API 및 CLI 명령 비교
{: #comparing-api-and-cli-commands-for-security-groups}

|설명 | API | CLI |
|-------------|-----|-----|
|URL 경로에서 ID로 지정된 VPC에 대한 기본 보안 그룹을 검색하십시오. | GET /vpcs/{vpc_id}/default_security_group| |
|이 모든 계정의 기존 보안 그룹을 검색하십시오.|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|URL 경로에서 ID로 지정된 단일 보안 그룹을 검색하십시오. | GET /security_groups/{id}| `ibmcloud is security-group`, `sg`|
|보안 그룹 템플리트에서 새 보안 그룹을 작성하십시오. 템플리트는 검색된 보안 그룹과 동일한 방법으로 구조화됩니다. 새 보안 그룹을 작성하는 데 필요한 정보가 포함되어 있습니다. 템플리트에는 그룹에 추가할 보안 그룹 규칙의 선택적 배열이 포함되어 있을 수 있습니다. | POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|서버의 네트워크 인터페이스에 연결된 기존 보안 그룹이 지정된 서버 인스턴스를 작성하십시오. |POST /instances(보안 그룹 매개변수 포함)| `ibmcloud is instance-create` |
|보안 규칙 패치 오브젝트에 제공된 정보로 보안 그룹을 업데이트하십시오. 보안 그룹 패치 오브젝트는 검색된 보안 그룹과 동일한 방법으로 구조화됩니다. 업데이트할 정보만 포함합니다. | PATCH /security_groups/{id}| `ibmcloud is security-group-update`, `sgu`|
|특정 보안 그룹에 대한 모든 규칙을 검색하십시오. | GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|URL 경로에서 ID로 지정된 단일 보안 그룹 규칙을 검색하십시오. | GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`, `sg-rule`|
|보안 그룹 규칙 템플리트에서 새 보안 그룹 규칙을 작성하십시오. 규칙 템플리트 오브젝트는 검색된 보안 그룹 규칙과 동일한 방법으로 구조화됩니다. 규칙을 작성하는 데 필요한 정보가 포함되어 있습니다. 규칙은 보안 그룹의 모든 네트워킹 인터페이스에 적용됩니다. | POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|규칙 패치 오브젝트에 제공된 정보로 보안 그룹 규칙을 업데이트하십시오. 패치 오브젝트는 검색된 보안 그룹 규칙과 동일한 방법으로 구조화됩니다. 업데이트할 정보만 포함되어야 합니다. | PATCH /security_groups/{security_group_id}/rules/{id}| |
|보안 그룹 규칙을 삭제하십시오. 이를 삭제해도 해당 규칙에 의해 허용된 기존 연결이 종료되지는 않습니다. 이 오퍼레이션은 되돌릴 수 없습니다. | DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|보안 그룹을 삭제하십시오. VPC용 기본 보안 그룹이거나 다른 보안 그룹에 원격으로 이를 참조하는 규칙이 포함된 경우 보안 그룹을 삭제할 수 없습니다. 이 오퍼레이션은 되돌릴 수 없습니다. | DELETE /security_groups/{id}| `ibmcloud is security-group-delete`, `sgd`|
|서버의 기존 네트워크 인터페이스를 기존 보안 그룹에 추가하십시오. 이 기능은 해당 인터페이스에 그룹 규칙을 적용합니다. 이는 인터페이스 간에 이러한 규칙을 따르는 새 연결이 작성되도록 허용합니다.<br /><br />또한 이 네트워크 인터페이스의 경우 이제 이 그룹을 참조하는 원격 그룹의 규칙에 의해 지정된 액세스가 허용됩니다.<br /><br />아래 제한사항을 참조하십시오.  | PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|보안 그룹에서 서버의 네트워크 인터페이스를 제거하십시오. 그룹의 규칙은 해당 인터페이스에서 새 네트워크 연결을 허용하는 데 더 이상 사용되지 않습니다. <br /><br />또한 이 네트워크 인터페이스의 경우 이 그룹을 참조하는 원격 그룹의 규칙에 의해 지정된 액세스가 허용됩니다. <br /><br />이 그룹의 멤버십에 의해 허용된 기존 연결은 종료되지 않습니다.| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|보안 그룹과 연관된 모든 네트워크 인터페이스를 검색하십시오. 이는 그룹에서 적용되는 규칙에 대한 네트워크 인터페이스입니다. | GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|URL 경로에서 ID로 지정된 단일 네트워크 인터페이스를 검색하십시오. 네트워크 인터페이스는 보안 그룹의 기존 멤버여야 합니다. | GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## ACL용 API 및 CLI 명령 비교
{: #comparing-api-and-cli-commands-for-acls}

|설명 | API | CLI |
|-------------|-----|-----|
|ACL 작성 |`POST /network_acls` | `ibmcloud is network-acl-create`|
|모든 ACL 검색 |`GET /network_acls` | `ibmcloud is network-acls`|
|특정 ACL 검색| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|특정 ACL 업데이트| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|특정 ACL 삭제| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|ACL 규칙 작성| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|ACL의 모든 규칙 검색| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|특정 ACL 규칙 검색| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|특정 ACL 규칙 업데이트| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|특정 ACL 규칙 삭제| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|서브넷에 ACL 연결|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
