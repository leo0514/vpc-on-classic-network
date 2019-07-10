---


copyright:
  years: 2018, 2019
lastupdated: "2019-06-12"

keywords: load balancer, public, listener, back-end, front-end, pool, round-robin, weighted, connections, methods, policies, APIs, access, ports

subcollection: vpc-on-classic-network

---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# VPC용 로드 밸런서 사용
{: #--using-load-balancers-in-ibm-cloud-vpc}

{{site.data.keyword.cloud}} **VPC용 로드 밸런서** 서비스는 VPC의 동일한 지역 내에 있는 다중 서버 인스턴스 간에 트래픽을 분배합니다.

## 공용 로드 밸런서
{: #public-load-balancer}

로드 밸런서 서비스 인스턴스에는 공용으로 액세스 가능한 완전한 도메인 이름(FQDN)이 지정됩니다. 이 도메인 이름은 VPC용 IBM Cloud Load Balancer 뒤에서 호스팅되는 애플리케이션에 액세스하기 위해 사용되어야 합니다. 이 도메인 이름은 하나 이상의 공인 IP 주소로 등록될 수 있습니다.

시간 경과에 따라 유지보수 및 스케일링 활동으로 인해 이러한 공인 IP 주소와 공인 IP 주소의 수가 변경될 수 있습니다. 사용자의 애플리케이션을 호스트하는 백엔드 서버 인스턴스(VSI)는 동일한 VPC 아래의 동일한 지역에서 실행되어야 합니다.

## 사설 로드 밸런서
{: #private-load-balancer}

사설 로드 밸런서는 동일한 지역 및 VPC 내의 사설 서브넷의 내부 클라이언트에만 액세스할 수 있습니다. 사설 로드 밸런서는 [RFC1918 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://tools.ietf.org/html/rfc1918){: new_window} 주소 공간으로부터의 트래픽만 허용합니다.

공용 로드 밸런서와 마찬가지로, 사설 로드 밸런서 서비스 인스턴스에는 완전한 도메인 이름(FQDN)이 지정됩니다. 그러나 이 도메인 이름은 하나 이상의 사설 IP 주소로 등록됩니다.

IBM Cloud 오퍼레이션이 시간 경과에 따라 유지보수 및 스케일링 활동을 기반으로 지정된 사설 IP 주소의 수와 값을 변경할 수 있습니다. 사용자의 애플리케이션을 호스트하는 백엔드 서버 인스턴스(VSI)는 동일한 VPC 아래의 동일한 지역에서 실행되어야 합니다.

기능의 요약 비교는 다음 표를 참조하십시오.

|기능 |공용 로드 밸런서 |사설 로드 밸런서 |
|--------|-------|-------|
|인터넷에서 액세스 가능 여부 |예, FQDN 사용 인터넷 |아니오, 동일한 지역 및 VPC의 내부 클라이언트만 |
|모든 트래픽 허용 여부 |예 |아니오, RFC 1918만 |
|사설 IP 수 |시간 경과에 따라 변경됨 |시간 경과에 따라 변경됨 |
|도메인 이름 등록 방법 |공인 IP 주소 |사설 IP 주소 |

## 프론트 엔드 리스너 및 백엔드 풀
{: #front-end-listeners-and-back-end-pools}

최대 10개의 프론트 엔드 리스너(애플리케이션 포트)를 정의하고 백엔드 애플리케이션 서버의 각 백엔드 풀에 맵핑할 수 있습니다. 로드 밸런서 서비스 인스턴스 및 프론트 엔드 리스너 포트에 지정된 완전한 도메인 이름(FQDN)은 공용 인터넷에 공개됩니다. 입력되는 사용자 요청은 이러한 포트에서 수신됩니다.

지원되는 프론트 엔드 리스너 프로토콜은 다음과 같습니다.
* HTTP
* HTTPS
* TCP

지원되는 백엔드 풀 프로토콜은 다음과 같습니다.
* HTTP
* TCP

입력 HTTPS 트래픽은 백엔드 서버와의 일반 텍스트 HTTP 통신을 허용하기 위해 로드 밸런서에서 종료됩니다.

최대 50개의 서버 인스턴스를 벡엔드 풀에 연결할 수 있습니다. 연결된 각 서버 인스턴스에는 포트가 구성되어야 합니다. 포트는 프론트 엔드 리스너 포트와 같거나 같지 않을 수 있습니다.

포트 범위 56500에서 56520까지는 관리 용도로 예약됩니다. 이러한 포트는 프론트 엔드 리스너 포트로 사용될 수 없습니다.
{: note}

## 계층 7 로드 밸런싱
{: #layer-7-load-balancing}

공용 및 사설 로드 밸런서 모두 계층 7 로드 밸런싱을 지원합니다. 구성된 정책 및 규칙을 기반으로 데이터 트래픽이 분배됩니다. _정책_은 수신 요청이 정책과 연관된 규칙과 일치하는 경우 트래픽이 분배되는 방법을 의미하는 조치를 정의합니다.

### 계층 7 정책
{: #layer-7-policy}

계층 7 정책은 리스너(HTTP 또는 HTTPS 리스너만)와 연관됩니다. 각 정책에는 규칙 세트가 있을 수 있습니다. 정책은 모든 규칙이 일치하는 경우**에만** 적용됩니다.

둘 이상의 정책을 리스너에 첨부할 수 있습니다. 일반적으로 우선순위가 가장 낮은 정책이 먼저 평가됩니다. 지정된 정책에 대한 우선순위는 고유해야 합니다.

수신 요청이 정책 규칙과 일치하지 않으면 기본 풀이 구성된 경우 요청이 리스너의 기본 풀로 경로 재지정됩니다.

다음은 계층 7 정책에 대해 지원되는 조치입니다.

* **거부:** 요청이 403 응답으로 거부됩니다.
* **경로 재지정:** 요청이 구성된 URL 및 응답 코드로 경로 재지정됩니다.
* **전달:** 요청이 특정 백엔드 풀로 전송됩니다.

우선순위에 관계없이 `reject`로 설정된 정책이 먼저 평가됩니다.

그 다음 `redirect`로 설정된 정책이 평가됩니다.

마지막으로 `forward`로 설정된 정책이 평가됩니다.

각 조치 카테고리 내에서 정책은 우선순위의 오름차순(가장 낮은 값에서 가장 높은 값의 순서)으로 평가됩니다.

### 계층 7 정책 특성
{: #layer-7-policy-properties}

특성 |설명
------------- | -------------
이름 |정책의 이름입니다. 이름은 리스너 내에서 고유해야 합니다. 
조치 |모든 정책 규칙이 일치하는 경우 수행할 조치입니다. 허용 가능한 값은 `reject`, `redirect` 및 `forward`입니다. 우선순위에 관계없이 `reject` 조치를 사용하는 정책이 항상 먼저 평가됩니다. `redirect` 조치를 사용하는 정책이 다음으로 평가되고, 이어서 `forward` 조치를 사용하는 정책이 평가됩니다. 
우선순위 |정책은 우선순위의 오름차순을 기준으로 평가됩니다. 
URL |조치가 `redirect`로 설정된 경우 요청이 경로 재지정될 URL입니다. 
HTTP 상태 코드 |조치가 `redirect`로 설정된 경우 로드 밸런서에서 리턴되는 응답의 상태 코드입니다. 허용 가능한 값은 301, 302, 303, 307 또는 308입니다. 
대상 |조치가 `forward`로 설정된 경우 요청이 전달되는 서버 인스턴스의 백엔드 풀입니다. 

### 계층 7 규칙
{: #layer-7-rules}

계층 7 규칙은 요청이 일치되는 방법을 정의합니다. 다음과 같은 세 가지 유형이 지원됩니다.

유형      |설명
----------| -----------------------
`hostname` |요청이 지정된 호스트 이름(예: `api.my_company.com`)과 일치됩니다. 
`header` |요청이 HTTP 헤더 필드(예: `Cookie`)와 일치됩니다. 
`path` |요청이 URL의 경로(예: `/index.html`)와 일치됩니다. 

요청을 일치시키려면 `condition`이 규칙에 정의되어야 합니다. 다음과 같은 세 가지 조건이 지원됩니다.

조건 |평가 유형 
----------------|---------------------
`contains` |추출된 필드에 제공된 문자열이 포함되어 있는지 확인합니다. 
`equals` |추출된 필드가 제공된 문자열과 동일한지 확인합니다. 
`matches_regex` |추출된 필드를 제공된 정규식과 일치시킵니다. 

## 계층 7 규칙 특성
{: #layer-7-rule-properties}

특성 |설명
------------- | -------------
유형 |규칙의 유형을 지정합니다. 허용 가능한 값은 `hostname`, `header` 또는 `path`입니다. 
조건 |규칙이 평가되는 조건을 지정합니다. 조건은 `contains`, `equals` 또는 `matches_regex`일 수 있습니다. 
필드 |HTTP 헤더 필드 이름을 지정합니다. 이 필드는 `header` 규칙 유형에만 적용 가능합니다. 예를 들어, HTTP 헤더의 쿠키를 일치시키려면 이 필드를 `Cookie`로 설정할 수 있습니다. 
값 |일치될 값입니다. 

## 로드 밸런싱 방법
{: #load-balancing-methods}

다음 세 가지 로드 밸런싱 방법으로 벡엔드 애플리케이션 서버 사이에 트래픽을 분배할 수 있습니다.

* **라운드 로빈:** 라운드 로빈은 기본 로드 밸런싱 방법입니다. 이 방법을 사용하면
로드 밸런서가 라운드 로빈 방식으로 입력 클라이언트 연결을 백엔드 서버로 전달합니다. 따라서 모든 백엔드 서버가
대략 동일한 수의 클라이언트 연결을 수신합니다.

* **가중치 라운드 로빈:** 이 방법을 사용하면
로드 밸런서가 해당 서버에 지정된 가중치에 비례하여 입력 클라이언트 연결을 백엔드 서버로 전달합니다. 각 서버에는
기본 가중치인 50이 지정되고 해당 가중치는 0에서 100 사이의 임의의 값으로 사용자 정의될 수 있습니다.

예를 들어 A, B, C라는 세 개의 애플리케이션 서버가 있으며 가중치가 각각 60, 60, 30으로 사용자 정의되어 있으면, 서버 A 및 B가 같은 수의 연결을 수신하는 반면 서버 C는 그 수의 반에 해당되는 연결을 수신합니다.

* **최소 연결:** 이 방법을 사용하면 지정된 시간에 가장 적은 수의 연결을 수행하는 서버 인스턴스가 다음 클라이언트 연결을 수신합니다.

**이러한 방법의 추가 특성:**

* 서버 가중치를 '0'으로 재설정하는 것은 해당 서버에 새 연결이 전달되지 않음을 의미합니다. 단, 모든 기존 트래픽은 계속 플로우됩니다. '0' 가중치를 사용하면 서버를 점진적으로 중단하고 서비스 회전에서 제거하는 데 도움이 됩니다.
* 서버 가중치 값은 가중치 라운드 로빈 방법을 사용하는 경우에만 적용할 수 있습니다. 라운드 로빈 및 최소 연결 로드 밴런싱 방법으로는 무시됩니다.

## 수평적 확장
{: #horizontal-scaling}

로드 밸런서는 로드에 따라 자동으로 해당 용량을 조정합니다. 이와 같은 조정이 발생하는 경우, 로드 밸런서의 DNS 이름과 연관된 IP 주소의 수가 변경되는 것을 확인할 수 있습니다.

## 상태 확인
{: #health-checks}

상태 확인 정의는 백엔드 풀에 필수입니다.

로드 밸런서는 정기적인 상태 확인을 수행하여 백엔드 포트의 상태를 모니터링하고 그 결과에 따라 클라이언트 트래픽을 전달합니다. 지정된 벡엔드 서버 포트가 비정상적인 것으로 판단되면 새 연결이 전달되지 않습니다. 로드 밸런서는 상태가 좋지 않은 포트를 계속 모니터링하여 상태가 다시 회복되면, 즉, 두 가지의 연속적인 상태 확인 시도를 통과하면 사용을 재개합니다.

HTTP 및 TCP 포트에 대한 상태 확인은 다음과 같이 수행됩니다.

* **HTTP:** 미리 지정된 URL에 대한 `HTTP GET` 요청이 백엔드 서버 포트에 전송됩니다. `200 OK` 응답을 수신하면 서버 포트의 상태가 양호한 것으로 표시됩니다. 기본 `GET` 상태 경로는 UI를 통한 "/"이며 이는 사용자 정의될 수 있습니다.

* **TCP:** 로드 밸런서가 지정된 TCP 포트에서 백엔드 서버와의 TCP 연결을 열려고 시도합니다. 연결 시도가 성공하면 서버 포트의 상태가 양호한 것으로 표시되고 연결이 닫힙니다.

기본 상태 확인 간격은 5초이며 상태 확인 요청에 대한 기본 제한시간은 2초이며 기본 재시도 수는 2입니다.
{: note}

## SSL 오프로딩 및 필수 권한
{: #ssl-offloading-and-required-authorizations}

모든 입력 HTTPS 연결의 경우, 로드 밸런서 서비스가 SSL 연결을 종료하고 백엔드 서버 인스턴스와의 일반 텍스트 HTTP 통신을 설정합니다. 이 기술을 사용하면 CPU 집중적인 SSL 핸드쉐이크 및 암호화 또는 복호화 태스크가 백엔드 서버 인스턴스에서 이동되므로 모든 CPU 사이클을 애플리케이션 트래픽 처리에 사용할 수 있습니다.

로드 밸런서가 SSL 오프로딩 태스크를 수행하려면 SSL 인증서가 필요합니다. [IBM 인증서 관리자![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}를 통해 SSL 인증서를 관리할 수 있습니다.

로드 밸런서에 SSL 인증서에 대한 액세스 권한을 부여하려면 로드 밸런서 서비스 인스턴스에 인증서 관리자 인스턴스에 대한 액세스를 부여하는 ** 서비스 간 권한 부여**를 사용으로 설정해야 합니다. [서비스 간 액세스 부여 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window} 문서에 따라 이러한 권한을 관리할 수 있습니다. **VPC 인프라**를 소스 서비스로 선택하고 **VPC용 로드 밸런서**를 리소스 유형으로 선택하고 **인증서 관리자**를 대상 서비스로 선택한 다음 **작성자** 서비스 액세스 역할을 지정하십시오. 

필수 권한이 제거되면, 로드 밸런서에 오류가 발생할 수 있습니다.
{: note}

## ID 및 액세스 관리(IAM)
{: #identity-and-access-management-iam}

**VPC용 로드 밸런서** 인스턴스에 대한 액세스 정책을 구성할 수 있습니다. 사용자 액세스 정책을 관리하려는 경우 ID 및 액세스 관리에 대한 자세한 정보는 [리소스에 대한 액세스 관리 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window}를 참조하십시오.

### 사용자에 대한 리소스 그룹 액세스 정책 구성
{: #configuring-resource-group-access-policies-for-users}

로드 밸런서를 작성하려면 리소스 그룹에 액세스해야 합니다. 로드 밸런서를 작성 중인 사용자는 제공된 리소스 그룹 또는 기본 리소스 그룹(제공된 리소스 그룹이 없는 경우)에 대한 적절한 액세스 권한이 필요합니다.

1. **관리 > 계정 > 사용자**로 이동하십시오. IBM Cloud 계정에 대한 액세스 권한이 있는 사용자 목록이 표시됩니다.
2. 액세스 정책을 지정할 사용자 이름을 선택하십시오. 사용자가 표시되지 않으면 **사용자 초대**를 클릭하여 사용자를 IBM Cloud 계정에 추가하십시오.
3. **액세스 권한 지정**을 선택하십시오.
4. **리소스 그룹 내의 액세스 권한 지정**을 선택하십시오.
5. **리소스 그룹** 드롭 다운 목록에서 원하는 리소스 그룹을 선택하십시오.
6. **리소스 그룹에 액세스 권한 지정** 드롭 다운 목록에서 원하는 액세스 권한을 선택하십시오.
7. **서비스** 드롭 다운 목록에서 원하는 서비스를 선택하십시오.
9. **지정**을 선택하여 사용자에게 리소스 그룹 액세스 정책을 지정하십시오.

### 사용자에 대한 리소스 액세스 정책 구성
{: #configuring-resource-access-policies-for-users}

| 플랫폼 액세스 역할 | 로드 밸런서 조치 |
|-------------|-----|
|관리자 | 로드 밸런서 작성/보기/편집/삭제 |
|편집자 | 로드 밸런서 작성/보기/편집/삭제 |
|뷰어 | 로드 밸런서 보기 |

1. **관리 > 계정 > 사용자**로 이동하십시오. IBM Cloud 계정에 대한 액세스 권한이 있는 사용자 목록이 표시됩니다.
2. 액세스 정책을 지정할 사용자 이름을 선택하십시오. 사용자가 표시되지 않으면 **사용자 초대**를 클릭하여 사용자를 IBM Cloud 계정에 추가하십시오.
3. **액세스 권한 지정**을 선택하십시오.
4. **리소스에 대한 액세스 권한 지정**을 선택하십시오.
5. **서비스** 드롭 다운 목록에서 **VPC 인프라**를 선택하십시오.
6. **리소스 유형** 드롭 다운 목록에서 **VPC용 로드 밸런서**를 선택하십시오.
7. **로드 밸런서 ID** 드롭 다운 목록에서 로드 밸런서 인스턴스 ID를 선택하거나 기본 값인 **모든 로드 밸런서**를 사용하십시오.
8. 사용자에게 플랫폼 액세스 역할을 지정하십시오.
9. **지정**을 선택하여 사용자에게 액세스 정책을 지정하십시오.

## Activity Tracker 통합
{: #activity-tracker-integration}

로드 밸런서 서비스가 **IBM Cloud Activity Tracker with LogDNA**와 통합됩니다. 이는 클라우드의 서비스 상태를 변경하는 사용자가 시작한 활동으로 트리거되며 CADF 표준을 준수하는 이벤트를 기록합니다.

로드 밸런서 서비스 인스턴스에서 감사 이벤트로 기록되는 자세한 조치 목록은 [Activity Tracker with LogDNA 이벤트](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers)를 참조하십시오.

모든 감사 이벤트는 `us-south` 지역의 "IBM Cloud Activity Tracker with LogDNA"에 기록됩니다. 로드 밸런서 서비스가 프로비저닝되는 지역은 중요하지 않습니다.
{:note}

이벤트를 보려면 사용자 계정으로 `us-south` 지역에 "IBM Cloud Activity Tracker with LogDNA" 인스턴스를 프로비저닝해야 합니다. 계정의 사용자에게 "IBM Cloud Activity Tracker with LogDNA" 인스턴스에 대한 **뷰어** 플랫폼 액세스 역할 및 **독자** 서비스 액세스 역할을 부여하는 IAM 정책이 있어야 합니다.

액세스 부여에 대한 자세한 정보는 [계정 이벤트를 볼 수 있는 권한 부여 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}에서 확인할 수 있습니다.

IBM Cloud 계정 사용자는 로드 밸런서 서비스에 대해 수행되는 계정 레벨 오퍼레이션을 모니터할 수 있습니다.
{: tip}

사용자 계정으로 "IBM Cloud Activity Tracker with LogDNA" 인스턴스를 프로비저닝하려면 다음 단계를 따르십시오.

1.  [IBM Cloud 콘솔에 로그인하십시오. ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://cloud.ibm.com/){: new_window}
2. 왼쪽 상단에 있는 ![메뉴 아이콘](../../icons/icon_hamburger.svg)을 클릭하십시오. 여기에서 **관찰 가능성 > Activity Tracker**를 선택하십시오.
3. 오른쪽 상단에서 **인스턴스 작성**을 클릭하십시오.
4. 서비스 이름을 정의하십시오.
5. `us-south`를 지역으로 선택하고 리소스 그룹을 선택하십시오.
6. 유료 계정이 있는 경우 `lite` 이외의 플랜을 선택하십시오.
7. **작성**을 클릭하십시오. 

다음은 **리스너 작성** 오퍼레이션에 대한 샘플 Activity Tracker 오퍼레이션입니다.
```bash
{
    "logSourceCRN": "crn:v1:bluemix:public:is:eu-gb:a/<ACCOUNT_ID>::load-balancer:4633518f-8aac-48a1-a694-d15ee6bd70e3",
    "meta": {
        "serviceProviderName": "is-load-balancer",
        "serviceProviderProjectId": "48a7a7b7-6642-4aa1-8af9-c1be4ef82050",
        "serviceProviderRegion": "ng",
        "userAccountIds": [
            <ACCOUNT_ID>
        ]
    },
    "payload": {
        "action": "is.load-balancer.load-balancer.listener.create",
        "eventTime": "2019-05-30T18:42:48.96+0000",
        "eventType": "activity",
        "id": "e4ee1906d01a35efe8bd8303ce0a734e",
        "initiator": {
            "credential": {
                "type": "token"
            },
            "host": {
                "address": <CLIENT_IP>,
                "agent": "python-requests/2.19.1"
            },
            "id": <USER-ID>,
            "name": <USER_ID>,
            "project_id": <ACCOUNT_ID>,
            "typeURI": "service/security/account/user"
        },
        "message": "is.load-balancer: create listener 4633518f-8aac-48a1-a694-d15ee6bd70e3 success",
        "observer": {
            "id": "activity-tracker.ng.bluemix.net",
            "name": "ActivityTracker",
            "typeURI": "security/edge/activity-tracker"
        },
        "outcome": "success",
        "reason": {
            "reasonCode": 201,
      "reasonType": "https://www.iana.org/assignments/http-status-codes/http-status-codes.xml"
        },
        "requestData": "{\"headers\":{\"RayID\":\"4df2d9911a3ac2bd\"},\"extraData\":{\"resourceName\":\"4633518f-8aac-48a1-a694-d15ee6bd70e3\"}}",
        "requestPath": "/v1/load_balancers/4633518f-8aac-48a1-a694-d15ee6bd70e3/listeners",
        "severity": "normal",
        "target": {
            "host": {
                "address": <API_END_POINT>
            },
            "id": "crn:v1:bluemix:public:is:eu-gb:a/<ACCOUNT_ID>::load-balancer:4633518f-8aac-48a1-a694-d15ee6bd70e3",
            "name": "4633518f-8aac-48a1-a694-d15ee6bd70e3",
            "typeURI": "/v1/load_balancers/4633518f-8aac-48a1-a694-d15ee6bd70e3/listeners"
        },
        "typeURI": "http://schemas.dmtf.org/cloud/audit/1.0/event"
    },
    "saveServiceCopy": true
}
```

## 사용 가능한 API
{: #lbaas-apis-available}

API 호출을 작성하려면 REST 클라이언트의 일부 양식을 사용해야 합니다. 예를 들어, `curl` 명령을 사용하여 모든 기존 로드 밸런서를 검색할 수 있습니다.

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

다음 절에서는 VPC 환경 내의 로드 밸런서에 대해 사용할 수 있는 API에 관한 세부사항을 제공합니다. 전체 스펙은 [VPC on Classic API 참조](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers)의 내용을 참조하십시오.

|설명 | API |
|-------------|-----|
| 로드 밸런서 작성 및 프로비저닝 | POST /load_balancers |
| 모든 로드 밸런서 검색 | GET /load_balancers |
| 로드 밸런서 검색 | GET /load_balancers/{id} |
| 로드 밸런서 삭제 | DELETE /load_balancers/{id} |
| 로드 밸런서 업데이트 | PATCH /load_balancers/{id} |
| 리스너 작성 | POST /load_balancers/{id}/listeners |
| 로드 밸런서의 모든 리스너 검색 | GET /load_balancers/{id}/listeners |
| 리스너 검색 | GET /load_balancers/{id}/listeners/{listener_id} |
| 리스너 삭제 | DELETE /load_balancers/{id}/listeners/{listener_id} |
| 리스너 업데이트 | PATCH /load_balancers/{id}/listeners/{listener_id} |
| 풀 작성 | POST /load_balancers/{id}/pools |
| 로드 밸런서의 모든 풀 검색 | GET /load_balancers/{id}/pools |
| 풀 검색 | GET /load_balancers/{id}/pools/{pool_id} |
| 풀 삭제 | DELETE /load_balancers/{id}/pools/{pool_id} |
| 풀 업데이트 | PATCH /load_balancers/{id}/pools/{pool_id} |
| 멤버 작성 | POST /load_balancers/{id}/pools/{pool_id}/members |
| 풀의 모든 멤버 검색 | GET /load_balancers/{id}/pools/{pool_id}/members |
| 멤버 검색 |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 풀에서 멤버 삭제 | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 멤버 업데이트 | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 풀의 멤버 업데이트 | PUT /load_balancers/{id}/pools/{pool_id}/members |
| 로드 밸런서의 통계 검색 | GET /load_balancers/{id}/statistics |
| 리스너의 모든 정책 검색 | GET /load_balancers/{id}/listeners/{listener_id}/policies
| 리스너에 대한 정책 작성 | POST /load_balancers/{id}/listeners/{listener_id}/policies
| 리스너에서 정책 삭제 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 리스너의 정책 검색 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 리스너의 정책 업데이트 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 정책과 연관된 모든 규칙 검색 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| 정책에 대한 규칙 작성 | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| 정책에서 규칙 삭제 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| 정책에서 규칙 검색 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| 정책의 규칙 업데이트 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## 로드 밸런서 예제
{: #load-balancer-example}

다음 예제에서는 API를 사용하여 포트 `80`에서 청취 중인 웹 애플리케이션을 실행하는 2 VPC 서버 인스턴스(`192.168.100.5` 및 `192.168.100.6`)의 앞에 로드 밸런서를 작성합니다. 로드 밸런서에는 프론트 엔드 리스너가 있으며 이로 인해 HTTPS를 통해 안전하게 웹 애플리케이션에 액세스할 수 있습니다. 그런 다음 API를 사용하여 로드 밸런서 인스턴스가 작성된 후의 세부사항을 가져올 수 있으며 로드 밸런서 인스턴스를 삭제할 수 있습니다.

### 예제 단계
{: #lbaas-example-steps}

다음 예제 단계에서는 [IBM Cloud UI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 또는 [VPC on Classic API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)를 사용하여 VPC, 서브넷 및 인스턴스를 프로비저닝하는 전제조건 단계를 건너뜁니다.

[CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference)를 사용하여 로드 밸런서 예제 단계를 실행할 수도 있습니다.
{: note}

**1단계. 리스너, 풀 및 연결된 서버 인스턴스(풀 멤버)가 있는 로드 밸런서 작성**

```bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

샘플 출력:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

다음 단계에서 사용할 로드 밸런서의 ID를 저장하십시오. 예를 들어, `lbid` 변수에 저장하십시오.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**2단계: 로드 밴런서 가져오기**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

프로비저닝을 위해서는 약간의 시간이 필요합니다. 로드 밸런서가 준비되면 `online` 및 `active` 상태가 되며, 다음 샘플 출력이 표시됩니다.

샘플 출력:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

**3단계: 로드 밴런서 삭제**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## 계층 7 예제: 정책 및 규칙 작성
{: #layer-7-examples-create-policy-and-rules}

다음 두 예제에서는 정책 및 규칙이 작성되고 리스너와 연관되는 방법을 보여주는 단계를 제공합니다. 

### 예제 1: 정책 및 규칙이 있는 HTTPS 리스너 작성. 정책은 `Redirect` 조치를 사용하여 작성됩니다.

```
bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers/$lbId/listeners" \
    -d '{
            "certificate_instance": {
                "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/1111111111111111111111111111:22222222-3333-4444-5555-666666666666:certificate:77777777777777777777777777777777"
            },
            "connection_limit": 2000,
            "port": 443,
            "protocol": "https",
            "policies": [
                {
                    "name": "hostname_header",
                    "action": "redirect",
                    "priority": 1,
                    "target": {
                        "url": "https://www.examples.com/",
                        "http_status_code": 307
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        },
                        {
                            "condition": "equals",
                            "type": "hostname",
                            "value": "abc.com"
                        }
                    ]
                },
                {
                    "name": "header_cookie",
                    "action": "redirect",
                    "priority": 5,
                    "target": {
                        "url": "https://www.mycookies.com/",
                        "http_status_code": 302
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        },
                        {
                            "condition": "equals",
                            "type": "header",
                            "field": "cookie",
                            "value": "flavor=oatmeal"
                        }
                    ]
                },
                {
                    "name": "path_hostname",
                    "action": "redirect",
                    "priority": 10,
                    "target": {
                        "url": "https://www.myexamples.com/",
                        "http_status_code": 301
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "hostname",
                            "value": "abc"
                        },
                        {
                            "condition": "equals",
                            "value": "/test",
                            "type": "path"
                          }
                    ]
                }
            ]
        }'
```
{: codeblock}

### 예제 2: 풀로 전달하는 조치(`Forward`)를 사용하는 정책을 작성하여 기존 리스너와 연관
```
bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers/$lbId/listeners/$listenerId/policies" \
    -d '{
            "policies": [
                {
                    "action": "forward",
                    "priority": 1,
                    "target": {
                        "id": "7df616da-4dd6-43d3-881d-801ae29e29fe"
                    },
                    "rules": [
                        {
                            "condition": "equals",
                            "type": "header",
                            "field": "cookie",
                            "value": "flavor=oatmeal"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 5,
                    "target": {
                        "id": "8061c411-0d50-4c79-b475-102666796434"
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 10,
                    "target": {
                        "id": "62914e09-3928-4d89-b7f7-1bb7a6d7fe85"
                    },
                    "rules": [
                        {
                            "condition": "matches_regex",
                            "type": "hostname",
                            "value": "abc[a-z]*.com"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 6,
                    "target": {
                        "id": "62914e09-3928-4d89-b7f7-1bb7a6d7fe85"
                    },
                    "rules": [
                        {
                            "condition": "equals",
                            "type": "path",
                            "value": "/test/testtest"
                        }
                    ]
                }
            ]
        }'
```
{: codeblock}

## FAQ
{: #load-balancer-faqs}

이 절에서는 **VPC용 로드 밸런서** 서비스에 대해 자주 묻는 몇 가지 질문에 대한 답변을 소개합니다.

### 내 로드 밸런서에 다른 DNS 이름을 사용할 수 있습니까?
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

로드 밸런서에 자동 지정된 DNS 이름은 사용자 정의할 수 없습니다. 그러나 선호하는 DNS 이름을 가리키는 CNAME(표준 이름) 레코드를 자동 지정된 로드 밸런서 DNS 이름에 추가할 수 있습니다. 예를 들어, `us-south`의 로드 밸런서 ID가 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`이며 자동 지정된 로드 밸런서 DNS 이름이 `dd754295-us-south.lb.appdomain.cloud`입니다. 선호하는 DNS 이름은 `www.myapp.com`입니다. `myapp.com`을 관리하기 위해 사용하는 DNS 제공자를 통해 `www.myapp.com`을 가리키는 CNAME 레코드를 로드 밸런서 DNS 이름 `dd754295-us-south.lb.appdomain.cloud`에 추가할 수 있습니다.

### 내 로드 밸런서로 정의할 수 있는 프론트 엔드 리스너는 최대 몇 개입니까?
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10.

### 내 백엔드 풀에 연결할 수 있는 서버 인스턴스는 최대 몇 개입니까?
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50.

### 로드 밸런서를 수평으로 규모 조절할 수 있습니까?
{: #is-the-load-balancer-horizontally-scalable}

예. 로드 밸런서는 로드에 따라 자동으로 해당 용량을 조정합니다. 수평적 확장이 발생하는 경우 로드 밸런서의 DNS 이름과 연관된 IP 주소의 수가 변경됩니다.

### 로드 밸런서를 배치하기 위해 사용되는 서브넷에서 ACL 또는 보안 그룹을 사용 중인 경우, 필요한 작업은 무엇입니까?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

구성된 리스너 포트 및 관리 포트(56500 - 56520 사이의 포트)에 대한 입력 트래픽을 허용하기 위해 적절한 ACL 또는 보안 그룹 규칙이 적소에 배치되었는지 확인해야 합니다. 로드 밸런서 및 백엔드 인스턴스 간의 트래픽도 허용되어야 합니다.

### `인증서 인스턴스를 찾을 수 없음`이라는 오류 메시지가 표시되는 이유는 무엇입니까?

* 인증서 인스턴스 CRN이 올바르지 않을 수 있습니다.
* **서비스 간 권한**을 부여하지 않았을 수 있습니다. 이 문서의 **SSL 오프로딩** 절을 참조하십시오.

### `401 Unauthorized Error` 코드를 수신하는 이유는 무엇입니까?

사용자에 대한 다음 액세스 정책을 확인하십시오.
* 로드 밸런서 리소스 유형에 대한 액세스 정책
* 리소스 그룹에 대한 액세스 정책
* `HTTPS` 리스너가 사용되는 경우 인증서 관리자 인스턴스에 대한 서비스 간 권한 부여도 확인하십시오.

### 로드 밸런서가 `maintenance_pending` 상태인 이유는 무엇입니까?

로드 밸런서는 다음과 같은 다양한 유지보수 활동 동안 `maintenance_pending` 상태가 됩니다.
* 수평적 확장 활동
* 복구 활동
* 취약점을 처리하고 보안 패치를 적용하기 위한 롤링 업그레이드

### 프로비저닝 동안 다중 서브넷을 선택해야 하는 이유는 무엇입니까?
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

**VPC용 로드 밸런서**에는 다중 구역 지역(MZR)이 준비되어 있습니다. 로드 밸런서 어플라이언스는 사용자가 선택한 서브넷에 배치됩니다. 고가용성 및 중복성을 제공하기 위해 다른 구역의 서브넷을 선택하도록 강력히 권장합니다.

### 내 풀 아래의 백엔드 멤버 상태가 `unknown`인 이유는 무엇입니까?

* 풀이 리스너와 연관되어 있지 않습니다.
* 풀 또는 풀과 연관된 리스너에 구성 변경사항이 작성되었을 수 있습니다.

### SSL 오프로드를 사용하여 지원되는 TLS 버전은 무엇입니까?
{: #which-tls-version-is-supported-with-ssl-offload}

**VPC용 로드 밸런서**는 SSL 종료를 사용하는 TLS 1.2를 지원합니다.

다음은 지원되는 암호에 대한 목록 세부사항입니다(우선순위에 따라 나열됨).
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### 상태 검사 매개변수에 대한 기본 설정 및 허용되는 값은 무엇입니까?
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

기본 설정 및 허용되는 값은 다음과 같습니다.
* 상태 검사 간격: 기본값은 5초이고 범위는 2 - 60초입니다.
* 상태 검사 응답 제한시간: 기본값은 2초이고 범위는 1 - 59초입니다.
* 최대 재시도 횟수: 기본값은 2회 재시도이고 범위는 1 - 10회 재시도입니다.

상태 검사 응답 제한시간 값은 항상 상태 검사 간격 값 미만이어야 합니다.
{:note}

### 로드 밸런서 IP 주소가 고정되어 있습니까?
{: #are-the-load-balancer-ip-addresses-fixed}

기본 제공되는 서비스 탄력성으로 인해 로드 밸런서 IP 주소가 고정되도록 보장되지 않습니다. 수평적 확장 중에 로드 밸런서의 FQDN과 연관된 사용 가능한 IP가 변경됩니다.

캐시된 IP 주소 대신 FQDN을 사용하십시오.
{:note}

### 로드 밸런서가 계층 7 스위칭을 지원합니까?
{: #does-the-load-balancer-support-layer-7-switching}

예.

### HTTPS 리스너 작성 또는 업데이트 시 내 인증서가 올바르지 않다는 알림을 받는 이유는 무엇입니까?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

다음 가능성을 확인하십시오.

* 제공된 인증서 CRN이 올바르지 않을 수 있습니다.
* 인증서 관리자에 지정된 인증서 인스턴스에 연관된 개인 키가 없을 수 있습니다.
