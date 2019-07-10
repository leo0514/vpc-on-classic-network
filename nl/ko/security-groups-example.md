---

copyright:
  years: 2019

lastupdated: "2019-05-14"

keywords: security group, VSI, ping, TCP, outbound, SG4, rules, metadata, setting up

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

# CLI를 사용하여 보안 그룹 설정
{: #setting-up-security-groups-using-the-cli}

다음 예제에서는 명령행 인터페이스(CLI)를 사용하여 보안 그룹을 사용하도록 설정하고 VSI를 작성합니다. 시나리오는 다음과 같습니다.

![IBM VPC의 보안 그룹](images/security-groups-schematic.svg "IBM VPC의 보안 그룹"){: caption="Figure: Security Groups for IBM VPC" caption-side="top"}

그림에서는 **SG4**라는 이름의 VSI에 내부 VPC 주소 `10.0.0.5` 외에 유동 IP `169.60.208.144`가 지정되어 있음을 볼 수 있으며 따라서 공용 인터넷에 연결할 수 있습니다. VSI **SG4**에 지정된 보안 그룹의 이름은 "demosg"입니다.

이름이 **SG8**로 지정된 VSI는 사설 IP 주소를 사용하여 VPC에 대해 내부적으로만 사용됩니다. VSI **SG8**에 지정된 보안 그룹의 이름은 "my_vpc_sg"입니다. 이러한 VSI 둘 다 `sgvpc`라는 이름의 VPC 및 동일한 서브넷 `10.0.0.0/28`에 있으므로 서로 통신할 수 있습니다.

## 보안 그룹이 연결된 VSI를 작성하기 위한 단계
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

"my_vpc_sg"에 대한 보안 그룹 규칙에는 기본 기능인 SSH, PING 및 아웃바운드 TCP가 포함됩니다.

`ibmcloud is sgc` 명령으로 보안 그룹을 먼저 작성한 다음 새로 작성한 이 보안 그룹을 포함할 VSI를 작성해야 합니다.

이 예제 코드에서는 몇 단계를 건너뜁니다. 아래에서 자세한 정보를 찾을 수 있습니다.

 * VPC 및 서브넷 작성에 대한 지시사항은 [VPC 작성](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 주제에 나와 있습니다.

이 예제 CLI 코드에서 명령을 복사하여 붙여넣는 방법으로 보안 그룹이 있는 VSI 작성을 시작할 수 있습니다. 이 샘플 코드에서 시스템 응답은 모두 표시되지 않습니다. _VPC_, _서브넷_, _이미지_ 및 _키_에 대한 올바른 리소스 ID 및 올바른 _보안 그룹 ID 번호_로 명령을 업데이트해야 합니다.

1. 보안 그룹 "my_vpc_sg" 작성

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   나중에 사용할 수 있도록 ID를 변수에 저장하십시오(예: `sg`).

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. SSH, PING 및 아웃바운드 TCP를 허용하는 규칙 추가

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. 새로 작성된 보안 그룹으로 VSI 작성

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \ 
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## 명령 목록 치트 시트
{: #command-list-cheat-sheet}

보안 그룹에 대해 사용 가능한 VPC CLI 명령의 전체 목록을 보려면 다음을 입력하십시오.

```
ibmcloud is help | grep sg
```
{: pre}

보안 그룹 및 규칙을 포함한 메타데이터를 보려면 다음을 입력하십시오(앞의 예의 경우).

```
ibmcloud is sg $sg
```
{: pre}

보안 그룹 규칙을 추가하려면 보안 그룹에 PING 인바운드 규칙을 추가하는 다음 예제 명령을 참조하십시오.

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
