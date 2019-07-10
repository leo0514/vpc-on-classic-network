---

copyright:
  years: 2018, 2019

lastupdated: "2019-05-14"

keywords: default, security group, asynchronous, rules

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

# 기본 보안 그룹 업데이트
{: #updating-the-default-security-group}


기본 보안 그룹은 삭제될 수 없다는 점만 제외하면 기타 보안 그룹과 매우 유사합니다.

각 VPC에는 다음을 허용하는 데 필요한 규칙이 있는 기본 보안 그룹이 있습니다.

* 모든 그룹 멤버로부터의 인바운드 트래픽
* 모든 아웃바운드 트래픽

기본 보안 그룹의 규칙을 편집하면 편집된 해당 규칙이 그룹 내의 모든 현재 및 미래 서버에 적용됩니다.

Ping 및 SSH를 허용하기 위한 인바운드 규칙은 기본 보안 그룹에 자동으로 추가되지 않습니다. REST API, `ibmcloud cli` 또는 UI를 사용하여 기본 보안 그룹의 규칙을 수정할 수 있습니다.

## 예: CLI를 사용하여 기본 보안 그룹 규칙 수정
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. IBM Cloud에 로그인하십시오.

   연합 계정이 있는 경우:
   ```
ibmcloud login -sso
   ```
   {: pre}

   그렇지 않으면 다음 명령을 사용하십시오.

   ```
ibmcloud login
   ```
   {: pre}

2. 기본 보안 그룹 ID 및 VPC에 대한 세부사항을 가져오십시오.

   모든 VPC를 나열하려면 다음 CLI 명령을 실행하십시오.

   ```
   ibmcloud is vpc
   ```
   {: pre}

   기본 보안 그룹 이름은 `기본 보안 그룹` 열 아래에 표시됩니다. 보안 그룹(다음)을 나열할 때 `ID`를 찾을 수 있도록 이름을 기록해 두십시오. 
   
   이제 VPC의 모든 보안 그룹을 나열하십시오.

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   나중에 사용할 수 있도록 보안 그룹 ID(기본 보안 그룹용)를 변수에 저장하십시오. 예를 들어 변수 이름 `sg`를 사용합니다.

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   보안 그룹에 대한 세부사항을 가져오려면 다음 CLI 명령을 실행하십시오.

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   또는 변수 `$sg` 대신에 실제 ID 값을 삽입할 수 있습니다.

3. 기본 보안 그룹 업데이트 -- SSH 및 PING을 허용하는 규칙 추가

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


보안 그룹 규칙 추가 및 제거는 비동기 작업입니다. 변경사항이 적용되려면 1 - 30초 정도가 걸립니다.
{: note}
