---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: security groups, RIAS, API, delete, create

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# API를 사용하여 보안 그룹 설정
{: #setting-up-security-groups-using-the-apis}

다음은 {{site.data.keyword.cloud}} VPC API를 사용하여 보안 그룹을 작성하고 관리하는 방법을 보여주는 예제입니다.

## 전제조건
{: #security-group-prerequisites}

보안 그룹을 사용하려면 먼저 실행 중인 {{site.data.keyword.cloud_notm}} VPC가 있어야 합니다.

VPC 및 서브넷 작성에 대한 지시사항은 [VPC 작성](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) 주제에 나와 있습니다.

## 1단계: 보안 그룹 작성
{: #step-1-create-a-security-group}

{{site.data.keyword.cloud_notm}} VPC에서 `my-security-group`이라는 보안 그룹을 작성하십시오.

```
curl -X POST "$rias_endpoint/v1/security_groups?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

나중에 사용할 수 있도록 ID를 변수에 저장하십시오(예: `sg`).

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## 2단계: SSH 연결 허용을 위해 규칙 추가
{: #step-2-add-a-rule-to-allow-ssh-connections}

포트 22에서 인바운드 연결을 허용할 보안 그룹에 대한 규칙을 작성하십시오.

```
curl -X POST "$rias_endpoint/v1/security_groups/$sg/rules?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```
{: pre}

## 3단계: 보안 그룹 삭제(선택사항)
{: #step-3-delete-the-securiy-group-optional}

보안 그룹을 정리하려면, 연관된 네트워크 인터페이스가 없어야 하며 다른 보안 그룹 내의 규칙에서 참조되지 않아야 합니다.

```
curl -X DELETE "$rias_endpoint/v1/security_groups/$sg?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token"
```
{: pre}
