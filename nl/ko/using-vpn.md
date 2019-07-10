---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-06-04"

keywords: VPN, network, encryption, authentication, algorithm, IKE, IPsec, policies, gateway, auto-negotiation

subcollection: vpc-on-classic-network


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# VPC와 함께 VPN 사용
{: #--using-vpn-with-your-vpc}
[comment]: # (링크된 도움말 항목)

{{site.data.keyword.cloud}} VPC VPN 서비스를 사용하면 안전하게 사설 네트워크에 연결할 수 있습니다. VPN을 사용하여 VPC 및 온프레미스 사설 네트워크 또는 또 다른 VPC 간의 IPsec 사이트 대 사이트 터널을 설정할 수 있습니다.

현재 {{site.data.keyword.cloud}} VPC 릴리스에 대해서는 정책 기반의 라우팅만 지원됩니다.

## 기능
{: #vpn-features}

* IKEv1 및 IKEv2
* 인증 알고리즘: `md5`, `sha1`, `sha256`
* 암호화 알고리즘: `3des`, `aes128`, `aes256`
* DH(Diffie-Hellman) 그룹: 2, 5, 14
* IKE 협상 모드: 기본
* IPSec 변환 프로토콜: ESP
* IPSec 캡슐화 모드: 터널
* PFS(Perfect Forward Secrecy)
* 데드 피어 발견
* 라우팅: 정책 기반
* 인증 모드: 사전 공유 키
* 활성/대기 모드에서만 HA 지원

## 사용 가능한 API
{: #apis-available}

다음 절에서는 IBM Cloud VPC 환경 내의 VPN에 대해 사용할 수 있는 API에 관한 세부사항을 제공합니다. 세부사항은 [VPC REST API](https://{DomainName}/apidocs/vpc-on-classic#list-all-ike-policies) 페이지를 참조하십시오.

### VPN 게이트웨이 및 VPN 연결
{: #vpn-gateways-and-vpn-connections}

|설명 | API |
|----------------------------|-------------|
| VPN 게이트웨이 작성 | POST /vpn_gateways |
| VPN 게이트웨이 검색 | GET /vpn_gateways |
| VPN 게이트웨이 검색 | GET /vpn_gateways/{id} |
| VPN 게이트웨이 삭제 | DELETE /vpn_gateways/{id} |
| VPN 게이트웨이 업데이트 | PATCH /vpn_gateways/{id} |
| 새 VPN 연결 작성 | POST /vpn_gateways/{vpn_gateway_id}/connections |
| VPN 연결 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections |
| VPN 연결 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 연결 삭제 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 연결 업데이트 | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 연결에 필요한 모든 로컬 CIDR 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| VPN 연결에서 로컬 CIDR 삭제 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 특정 로컬 CIDR이 VPN 연결에 존재하는지 확인 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 연결에 로컬 CIDR 설정 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 연결에 필요한 모든 피어 CIDR 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| VPN 연결에서 피어 CIDR 삭제 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| 특정 피어 CIDR이 VPN 연결에 존재하는지 확인 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| VPN 연결에 피어 CIDR 설정 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE 정책
{: #ike-policies}

|설명 | API |
|-----------------------------|--------------|
| 모든 IKE 정책 검색 | GET /ike_policies |
| IKE 정책 작성 | POST /ike_policies |
| IKE 정책 삭제 | DELETE /ike_policies/{id} |
| IKE 정책 검색 | GET /ike_policies/{id} |
| IKE 정책 업데이트 | PATCH /ike_policies/{id} |
| 지정된 IKE 정책을 사용하는 모든 연결 검색 | GET /ike_policies/{id}/connections |

### IPsec 정책
{: #ipsec-policies}

|설명 | API |
|---------------------|-------------|
| 모든 IPSec 정책 검색 | GET /ipsec_policies |
| IPSec 정책 작성 | POST /ipsec_policies |
| IPSec 정책 삭제 | DELETE /ipsec_policies/{id} |
| IPSec 정책 검색 | GET /ipsec_policies/{id} |
| IPSec 정책 업데이트 | PATCH /ipsec_policies/{id} |
| 지정된 IPsec 정책을 사용하는 모든 연결 검색 | GET /ipsec_policies/{id}/connections |

## VPN 예제
{: #vpn-example}

다음 예제에서는 VPN을 사용하여 두 개의 VPC에 함께 연결할 수 있습니다. 즉, 두 개의 별도의 VPC 내의 서브넷이 단일 네트워크에 존재하는 것처럼 연결할 수 있습니다. 서브넷의 IP 주소가 겹치지 않아야 합니다.
시나리오는 다음과 같습니다(각 VPC에 일부 VM이 추가됨).

![IBM VPC용 VPN](images/vpc-vpn.svg "IBM VPC용 VPN"){: caption="Figure: VPN for IBM VPC" caption-side="top"}

### 예제 단계
{: #vpn-example-steps}

다음 예제 단계에서는 IBM Cloud API 또는 CLI를 사용하여 VPC를 작성하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [시작하기](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) 및 [API로 VPC 설정](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)을 참조하십시오.

필요에 따라 UI를 사용하여 VPN 게이트웨이를 작성할 수 있습니다. 단계는 [콘솔 튜토리얼 문서](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)를 참조하십시오.

#### 1단계: VPC 서브넷에서 VPN 게이트웨이 작성
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

샘플 출력:
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

후속 단계를 위해 다음 필드를 저장하십시오.
* `id`. VPN 게이트웨이 ID이며 `$gwid1`로 참조됩니다.
* `address`. VPN 게이트웨이의 공인 IP 주소이며 `$gwaddress1`로 참조됩니다.

게이트웨이 상태는 VPN 게이트웨이가 작성 중인 동안 `pending`으로 표시되다가 작성이 완료되면 `available`로 변경됩니다. 작성에는 시간이 걸릴 수 있습니다.
{: note}


다음 명령을 사용하여 게이트웨이의 상태를 확인할 수 있습니다.

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### 2단계: 다른 VPC에서 두 번째 VPN 게이트웨이를 작성하십시오.
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

샘플 출력:
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

후속 단계를 위해 다음 필드를 저장하십시오.
* `id`. VPN 게이트웨이 ID이며 `$gwid2`로 참조됩니다.
* `address`. VPN 게이트웨이의 공인 IP 주소이며 `$gwaddress2`로 참조됩니다.


#### 3단계. 첫 번째 VPN 게이트웨이에서 두 번째 VPN 게이트웨이로의 VPN 연결 작성
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

연결을 작성할 때 `local_cidrs`를 **VPC 1**의 서브넷으로 설정하고 `peer_cidrs`를 **VPC 2**의 서브넷으로 설정하십시오.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

샘플 출력:
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 4단계. 두 번째 VPN 게이트웨이에서 첫 번째 VPN 게이트웨이로의 VPN 연결 작성
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

연결을 작성할 때 `local_cidrs`를 **VPC 2**의 서브넷으로 설정하고 `peer_cidrs`를 **VPC 1**의 서브넷으로 설정하십시오.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress1,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

샘플 출력:
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 5단계: 연결 확인
{: #step-5-verify-connectivity}

VPN 연결이 설정되고 나면 서브넷 1에서 서브넷 2의 인스턴스에 연결할 수 있으며 그 반대의 경우도 가능합니다.

다음과 같이 VPN 연결의 상태를 확인할 수 있습니다.
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

샘플 출력:
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## 할당량
{: #see-vpn-quotas}

VPN 할당량은 [VPC 할당량](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas) 주제를 참조하십시오.

## 정책 자동 협상
{: #policy-auto-negotiation}

VPN 연결을 구성하기 위해 IKE 및 IPsec 정책을 사용하는 것은 선택사항입니다. 정책을 선택하지 않으면 _자동 협상_이라 불리는 프로세스에 대한 기본 제안이 자동으로 선택됩니다. 

IBM Cloud 자동 협상은 **IKEv2**를 사용하므로 온프레미스 디바이스도 **IKEv2**를 사용해야 합니다. 온프레미스 디바이스가 **IKEv2**를 지원하지 않는 경우 사용자 정의된 IKE 정책을 사용하십시오.
{: note}

### IKE 자동 협상(1단계)
{: #ike-auto-negotiation-phase-1}

다음과 같은 암호화, 인증 및 DH(Diffie-Hellman) 그룹 옵션을 원하는 대로 조합하여 사용할 수 있습니다.

|    | 암호화 |인증 | DH 그룹 |
|----|------------|----------------|----------|
|1  | aes128 | sha1   |2  |
|2  | aes256 | sha256 |5  |
|3  | 3des   | md5    | 14 |

### IPsec 자동 협상(2단계)
{: #ipsec-auto-negotiation-phase-2}

다음과 같은 암호화 및 인증 옵션을 원하는 대로 조합하여 사용할 수 있습니다.

|    | 암호화 |인증 | DH 그룹 |
|----|------------|----------------|----------|
|1  | aes128 | sha1   | 사용 불가능  |
|2  | aes256 | sha256 | 사용 불가능  |
|3  | 3des   | md5    | 사용 불가능  |

## FAQ
{: #vpn-faq}

**VPN 게이트웨이를 작성할 때 동시에 VPN 연결을 작성할 수 있습니까?**

API 또는 CLI를 사용하는 경우, VPN 게이트웨이를 작성한 후에 VPN 연결을 작성해야 합니다. IBM Cloud 콘솔에서는 게이트웨이 및 연결을 동시에 작성할 수 있습니다.

**VPN 연결이 접속된 VPN 게이트웨이를 삭제한 경우, 연결은 어떻게 됩니까?**

VPN 연결도 VPN 게이트웨이와 함께 삭제됩니다.

**VPN 게이트웨이 또는 VPN 연결을 삭제하면 IKE 또는 IPSec 정책도 삭제됩니까?**

아니오, IKE 및 IPSec 정책은 다중 연결에 적용될 수 있습니다.

**게이트웨이가 위치한 서브넷을 삭제하려고 시도하면 VPN 게이트웨이는 어떻게 됩니까?**

VPN 게이트웨이를 포함하여 인스턴스가 있으면 서브넷을 삭제할 수 없습니다.

**기본 IKE 및 IPsec 정책이 있습니까?**

정책 ID(IKE 또는 IPsec)를 참조하지 않고 VPN 연결을 작성하면 자동 협상이 사용됩니다.

**VPN 게이트웨이 프로비저닝 중에 서브넷을 선택해야 하는 이유는 무엇입니까?**

서브넷은 VPN 게이트웨이를 VPC의 다른 리소스와 연결합니다. 권장되는 우수 사례는 서브넷에 사용 가능한 사설 IP가 충분하도록 이 서브넷에 다른 VPC 인스턴스 없이 VPN 게이트웨이 전용 서브넷을 작성하는 것입니다. VPN 게이트웨이에는 HA 및 롤링 업그레이드를 수용할 수 있도록 8개의 사설 IP 주소가 필요합니다.

**VPN 게이트웨이는 어떤 구역에 상주합니까?**

VPN 게이트웨이는 사용자가 프로비저닝 중에 선택한 서브넷이 있는 구역에 상주합니다. VPN 게이트웨이는 동일한 구역 및 동일한 VPC의 VPC 인스턴스만 제공한다는 점을 기억하십시오. 따라서 다른 구역의 VPC 인스턴스는 VPN 게이트웨이를 활용하여 온프레미스 사설 네트워크와 통신할 수 없습니다. 구역 결함 허용을 위해 구역당 하나의 VPN 게이트웨이를 배치해야 합니다.

**VPN 게이트웨이를 배치하는 데 사용되는 서브넷에서 ACL을 사용 중인 경우 필요한 작업은 무엇입니까?**

관리 트래픽 및 VPN 터널 트래픽을 허용하려면 다음 ACL 규칙이 있는지 확인하십시오.

* **인바운드 규칙**
    - TCP 프로토콜 소스 포트 9091 허용
    - TCP 프로토콜 소스 포트 10514 허용
    - TCP 프로토콜 소스 포트 443 허용
    - TCP 프로토콜 소스 포트 80 허용
    - TCP 프로토콜 소스 포트 53 허용
    - UDP 프로토콜 소스 포트 53 허용
    - ALL 프로토콜 소스 IP를 VPN 피어 게이트웨이 공인 IP로 허용
    - TCP 프로토콜 대상 포트 443 허용
    - TCP 프로토콜 대상 포트 56500 허용
    - VPC의 인스턴스와 온프레미스 사설 네트워크 간의 트래픽 허용
    - ICMP 트래픽 허용

* **아웃바운드 규칙**
   - 모든 트래픽 허용

**온프레미스 사설 네트워크와 통신해야 하는 서브넷에서 ACL 또는 보안 그룹을 사용 중인 경우 필요한 작업은 무엇입니까?**

VPC의 인스턴스와 온프레미스 사설 네트워크 간의 트래픽을 허용하기 위해 ACL 규칙 또는 보안 그룹 규칙이 적소에 배치되었는지 확인해야 합니다. 

**VPC용 VPN이 HA 구성을 지원합니까?**

예, 활성/대기 구성에서 HA를 지원합니다.

**SSL VPN을 지원할 계획이 있습니까?**

아니오, IPsec 사이트 간 VPN만 지원됩니다.

**사이트 간 VPNaaS에 대한 처리량에 상한이 있습니까?**

최대 650Mbps의 처리량이 지원됩니다.

**VPNaaS에 대해 PSK 및 인증서 기반 IKE 인증이 지원됩니까?**

PSK 인증만 지원됩니다.

**VPC용 VPN을 IBM Cloud 인프라 클래식에 대한 VPN 게이트웨이로 사용할 수 있습니까?**

아니오, IBM Cloud 인프라 클래식 환경에서 VPN 게이트웨이를 사용하려면 [IPsec VPN](https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn)을 사용해야 합니다.
