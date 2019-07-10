---

copyright:
  years: 2019

lastupdated: "2019-06-11"

keywords: ACLs, network, CLI, example, tutorial, firewall, subnet, inbound, outbound, rule

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


# 네트워크 ACL 설정
{: #setting-up-network-acls}
[comment]: # (링크된 도움말 항목)

{{site.data.keyword.cloud}} 가상 사설 클라우드에서 사용 가능한 액세스 제어 목록(ACL) 기능을 사용하면 클라우드의 중요한 비즈니스 워크로드와 관련된 모든 입력 및 출력 트래픽을 제어할 수 있습니다. ACL은 보안 그룹과 유사한 기본 제공 가상 방화벽입니다. 보안 그룹과 대조적으로 ACL 규칙은 _인스턴스_가 아닌 _서브넷_에 대한 트래픽을 제어합니다.

보안 그룹과 ACL 간의 특성 비교는 [비교 표](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists)를 참조하십시오.

이 문서에서 제공된 예제에서는 CLI를 사용하여 VPC에서 서브넷을 보호하는 네트워크 ACL을 작성하는 방법을 보여줍니다.

## ACL에 사용 가능한 API 및 CLI
{: #apis-and-clis-are-available-for-acls}

API, CLI 또는 UI를 통해 ACL을 설정하고 관리할 수 있습니다.

* 각 API에 대한 매개변수, 요청 본문, 응답 세부사항은 [API 참조](https://{DomainName}/apidocs/vpc-on-classic)를 참조하십시오.

* 명령 세부사항, CLI 플러그인 설치 단계, VPC CLI 사용을 위한 전제조건 단계를 얻으려면 이 [CLI 예제](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)를 참조하십시오.

* {{site.data.keyword.cloud_notm}} 콘솔에서 ACL을 설정하는 방법에 대한 정보는 [UI를 사용하여 ACL 구성](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-acl)을 참조하십시오.

ACL 규칙은 _서브넷_에 대한 트래픽을 제어합니다. VPC API, CLI 및 UI가 인바운드 규칙에 대한 **대상 IP 범위** 및 아웃바운드 연결에 대한 **소스 IP 범위**를 사용자 정의하기 위한 지원을 제공하는 것처럼 보이지만 이러한 주소는 사용자 정의할 수 없습니다. **인바운드 ACL 규칙의 대상 IP 범위 및 아웃바운드 ACL 규칙의 소스 IP 범위는 각각 연결된 서브넷에 연결되어 있습니다**. 따라서 실제로 **0.0.0.0/0**이 인바운드 규칙의 대상 IP 및 아웃바운드 규칙의 소스 IP로 사용되어야 합니다.
{: note}

## ACL 및 ACL 규칙 관련 작업
{: #working-with-acls-and-acl-rules}

ACL을 효과적으로 만들기 위해 인바운드 및 아웃바운드 네트워크 트래픽을 처리하는 방법을 판별하는 규칙을 작성합니다. 다중 인바운드 및 아웃바운드 규칙을 작성할 수 있습니다. 작성할 수 있는 규칙 수에 대한 특정 정보는 [할당량](/docs/vpc-on-classic?topic=vpc-on-classic-quotas)을 참조하십시오.

* 인바운드 규칙을 사용하면 지정된 프로토콜 및 포트를 사용하는 소스 IP 범위로부터의 트래픽을 허용하거나 거부할 수 있습니다. 대상 IP 범위는 연결된 서브넷에 따라 판별됩니다.
* 아웃바운드 규칙을 사용하면 지정된 프로토콜 및 포트를 사용하는 대상 IP 범위로의 트래픽을 허용하거나 거부할 수 있습니다. 소스 IP 범위는 연결된 서브넷에 따라 판별됩니다.
* ACL 규칙은 순서대로 고려됩니다. 우선순위 번호가 더 낮은(예: 1) 규칙이 일치하지 않는 경우에만 우선순위 번호가 더 높은(예: 2) 규칙이 평가됩니다.
* 인바운드 규칙은 아웃바운드 규칙과 별도입니다.
* 현재 지원되는 프로토콜은 TCP, UDP 및 ICMP입니다. 또한 **all** 옵션을 사용하여 _모든_ 프로토콜 또는 _기타_ 프로토콜(높은 우선순위의 규칙이 지정된 경우) 프로토콜을 지정할 수 있습니다.

ACL 규칙에서 ICMP, TCP 및 UDP 프로토콜을 사용하는 데 대한 관련 정보는 [인터넷 통신 프로토콜 이해](/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols)의 내용을 참조하십시오.

### 서브넷에 ACL 연결
{: #attaching-an-acl-to-a-subnet}

ACL을 서브넷에 연결하는 두 가지 옵션이 있습니다.

* 새 서브넷을 작성하고 연결할 ACL을 지정할 수 있습니다. ACL을 지정하지 않으면 기본 네트워크 ACL이 연결됩니다. 기본 ACL은 이 서브넷으로 향하는 모든 인바운드 트래픽 및 이 서브넷으로부터의 모든 아웃바운드 트래픽을 허용합니다.
* ACL을 기존 서브넷에 연결할 수 있습니다. 이 서브넷에 다른 ACL이 이미 연결되어 있으면 해당 ACL이 새 ACL이 연결되기 전에 분리됩니다.

## ACL 데모 예제
{: #acl-demo-example}

다음 예제에서는 명령행 인터페이스(CLI)를 사용하여 두 개의 ACL을 작성하고 두 개의 서브넷과 연결합니다. 시나리오는 다음과 같습니다.

![예제 ACL 시나리오](images/vpc-acls.png)

그림에서 보듯이 인터넷으로부터의 요청을 처리하는 두 개의 웹 서버와 공개하기를 원하지 않는 두 개의 백엔드 서버가 있습니다. 이 예제에서는 서버를 두 개의 독립된 서브넷, 즉, 10.10.10.0/24 및 10.10.20.0/24에 각각 배치하고 웹 서버가 백엔드 서버와 데이터를 교환할 수 있도록 허용해야 합니다. 또한 백엔드 서버에서 제한된 아웃바운드 트래픽을 허용할 수 있습니다.

### 예제 규칙
{: #acl-example-rules}

다음 예제 규칙은 앞에서 설명한 대로 기본 시나리오에 대한 ACL 규칙을 설정하는 방법을 표시합니다.

우수 사례로는 잘 세분화된 규칙에 덜 세분화된 규칙보다 높은 우선순위를 부여하는 것이 좋습니다. 예를 들어, 서브넷 10.10.30.0/24로부터의 모든 트래픽을 차단하는 규칙이 있으며 이 규칙이 더 높은 우선순위와 일치하는 경우, 10.10.30.5로부터의 트래픽을 허용하는 더 낮은 우선순위의 세분화된 규칙은 결코 적용되지 않습니다.
{:note}

**ACL-1 예제 규칙**:

| 인바운드/아웃바운드| 허용/거부 | 소스 IP | 대상 IP | 프로토콜 | 포트 |설명|
|--------------|-----------|------|------|------|------------------|-------|
| 인바운드 | 허용 | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 80 | 인터넷으로부터의 HTTP 트래픽 허용|
| 인바운드 | 허용 | 0.0.0.0/0 | 0.0.0.0/0 | TCP | 443 | 인터넷으로부터의 HTTPS 트래픽 허용|
| 인바운드 | 허용| 10.10.20.0/24 | 0.0.0.0/0 |모두|모두| 백엔드 서버가 배치된 서브넷 10.10.20.0/24로부터의 모든 인바운드 트래픽 허용|
| 인바운드 | 거부| 0.0.0.0/0| 0.0.0.0/0 |모두|모두| 모든 기타 인바운드 트래픽 거부|
| 아웃바운드 | 허용 | 0.0.0.0/0 | 0.0.0.0/0 | TCP|80 | 인터넷에 대한 HTTP 트래픽 허용|
| 아웃바운드 | 허용 | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 443 | 인터넷에 대한 HTTPS 트래픽 허용|
| 아웃바운드 | 허용| 0.0.0.0/0 | 10.10.20.0/24 | 모두|모두| 백엔드 서버가 배치된 서브넷 10.10.20.0/24에 대한 모든 아웃바운드 트래픽 허용|
| 아웃바운드 | 거부| 0.0.0.0/0 | 0.0.0.0/0|모두|모두| 모든 기타 아웃바운드 트래픽 거부|


**ACL-2 예제 규칙**:

| 인바운드/아웃바운드| 허용/거부 | 소스 IP | 대상 IP | 프로토콜| 포트 |설명|
|--------------|-----------|------|------|------|------------------|--------|
| 인바운드 | 허용| 10.10.10.0/24 | 0.0.0.0/0 |모두|모두| 웹 서버가 배치된 서브넷 10.10.10.0/24로부터의 모든 인바운드 트래픽 허용|
| 인바운드 | 거부| 0.0.0.0/0| 0.0.0.0/0 |모두|모두| 모든 기타 인바운드 트래픽 거부|
| 아웃바운드 | 허용 | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 80 | 인터넷에 대한 HTTP 트래픽 허용|
| 아웃바운드 | 허용 | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 443 | 인터넷에 대한 HTTPS 트래픽 허용|
| 아웃바운드 | 허용| 0.0.0.0/0 | 10.10.10.0/24 |모두|모두| 웹 서버가 배치된 서브넷 10.10.10.0/24에 대한 모든 아웃바운드 트래픽 허용|
| 아웃바운드 | 거부| 0.0.0.0/0 | 0.0.0.0/0|모두|모두| 모든 기타 아웃바운드 트래픽 거부|

이 예제에서는 일반 사례에 대해서만 설명합니다. 사용자의 시나리오에서는 트래픽을 좀 더 세분화하여 제어해야 하는 경우가 있을 수 있습니다.

* 운영 목적으로 원격 네트워크에서 10.10.10.0/24 서브넷에 대한 액세스가 필요한 네트워크 관리자가 있습니다. 이 경우, 인터넷에서 이 서브넷으로의 SSH 트래픽을 허용해야 합니다.
* 두 서브넷 사이에 허용할 프로토콜 범위를 좁히고자 할 수 있습니다.

### 예제 단계
{: #acl-example-steps}

다음 예제 단계에서는 VPC를 작성하기 위해 CLI를 사용하는, 먼저 수행되어야 하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [CLI를 사용하여 VPC 작성](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)의 내용을 참조하십시오.


#### 1단계: ACL 작성
{: #step-1-create-the-acls}

다음은 `my_web_subnet_acl` 및 `my_backend_subnet_acl`이라는 두 개의 ACL을 작성하기 위해 사용할 수 있는 CLI 명령입니다.

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

응답에는 새로 작성된 ACL ID가 포함됩니다. 이후 명령에서 사용할 두 ACL의 ID를 모두 저장하십시오. 다음과 같이 변수 `webacl` 및 `bkacl`을 사용할 수 있습니다.

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### 2단계: 기본 ACL 규칙 검색
{: #step-2-retrieve-the-default-acl-rules}

새 규칙을 추가하기 전에 그 앞에 새 규칙을 삽입할 수 있도록 기본 인바운드 및 아웃바운드 ACL 규칙을 검색하십시오.

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

응답은 모든 프로토콜에서 모든 IPv4 트래픽을 허용하는 기본 인바운드 및 아웃바운드 규칙을 표시합니다.

```
Getting rules of network acl ba9e785a-3e10-418a-811c-56cfe5669676 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

이후 명령에서 사용할 변수로 두 ACL 규칙의 ID를 모두 저장하십시오. 예를 들어 변수 `inrule` 및 `outrule`을 사용할 수 있습니다.

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### 3단계: 설명에 따라 새 ACL 규칙 추가
{: #step-3-add-new-acl-rules-as-decribed}

이 섹션에서는 먼저 인바운드 규칙을 추가한 후 아웃바운드 규칙을 추가하는 방법을 보여줍니다.

기본 인바운드 규칙 앞에 새 인바운드 규칙을 삽입하십시오.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

각 단계에서는 이후 명령에 사용될 ACL 규칙의 ID를 변수에 저장하십시오. 예를 들어 변수 이름 `acl200`을 사용할 수 있습니다.

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

이제 규칙을 `acl200`에 추가하십시오.

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

각 ID를 변수로 저장하여 ACL 설정이 완료될 때까지 규칙을 추가하십시오.

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

기본 아웃바운드 규칙 앞에 새 아웃바운드 규칙을 삽입하십시오.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e $webacl deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $outrule
acl200e="11110627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule100e $webacl allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule $acl200e
acl100e="22220627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20e $webacl allow outbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100e
acl20e="33330627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10e $webacl allow outbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20e
```
{: codeblock}

#### 4단계. 새로 작성된 ACL로 두 개의 서브넷 작성
{: #step-4-create-the-two-subnets-with-the-newly-created-acl}

각 ACL이 새 서브넷 중 하나와 연결되도록 두 개의 서브넷을 작성합니다.

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## 명령 목록 치트 시트
{: #acl-cli-command-list-cheat-sheet}

ACL에 대해 사용 가능한 VPC CLI 명령의 전체 목록을 보려면 다음을 사용하십시오.

```
ibmcloud is network-acls
```
{: pre}

ACL 및 규칙을 포함한 메타데이터를 보려면 다음을 사용하십시오.

```
ibmcloud is network-acl $webacl
```
{: pre}

기본 인바운드 ACL 규칙을 가져오려면 다음을 사용하십시오.

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## 예제 인바운드 `ping` 규칙
{: #acl-example-inbound-ping-rule}

ACL 규칙을 추가하려면 기본 인바운드 규칙 앞에 `ping` 인바운드 규칙을 추가하는 다음 예제 명령을 참조하십시오.

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
