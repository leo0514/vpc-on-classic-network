---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# 원격 Cisco ASAv 피어와의 보안 연결 작성
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

이 문서는 Cisco ASAv, Cisco Adaptive Security Appliance Software 버전 9.10(1)을 기반으로 합니다.

다음 예제 단계에서는 {{site.data.keyword.cloud}} API 또는 CLI를 사용하여 가상 사설 클라우드(VPC)를 작성하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [시작하기](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) 및 [API로 VPC 설정](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)을 참조하십시오.

## 예제 단계
{: #cisco-example-steps}

원격 Cisco ASAv 피어에 대한 연결의 토폴로지는 두 {{site.data.keyword.cloud}} 가상 사설 클라우드(VPC) 사이에 VPN 연결을 작성하는 것과 유사합니다. 단, 한 측이 Cisco ASAv 장치로 대체됩니다.

![여기에 이미지 설명 입력](./images/vpc-vpn-asav-figure.png)

### 원격 Cisco ASAv 피어와의 보안 연결 작성 방법
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

Cisco ASAv를 IBM VPC VPN과 함께 사용하도록 구성하는 첫 번째 단계는 다음 전제조건이 충족되는지 확인하는 것입니다.

* Cisco ASAv가 온라인 상태이며 적절한 라이센스로 작동 중입니다.
* Cisco ASAv에 대해 비밀번호가 사용되고 있습니다.
* 내부 인터페이스가 하나 이상 구성되어 있으며 작동 중임이 확인되었습니다.
* 외부 인터페이스가 하나 이상 구성되어 있으며 작동 중임이 확인되었습니다.

Cisco ASAv 장치가 원격 VPN 피어로부터 연결 요청을 수신하면 IPsec 1단계(Phase) 매개변수를 사용하여 보안 연결을 설정하고 해당 VPN 피어를 인증합니다. 그런 다음, 보안 정책이 연결을 허용하면 Cisco ASAv가 IPsec 2단계(Phase) 매개변수를 사용하여 터널을 설정하고 IPsec 보안 정책을 적용합니다. 키 관리, 인증 및 보안 서비스가 IKE 프로토콜을 통해 동적으로 협상됩니다.

**해당 기능을 지원하려면 Cisco ASAv 장치에 의해 다음과 같은 일반 구성 단계가 수행되어야 합니다.**

* Cisco ASAv 장치가 원격 피어를 인증하고 보안 연결을 설정하는 데 필요한 1단계(Phase) 매개변수를 정의하십시오.
* Cisco ASAv 장치가 원격 피어를 사용하여 VPN 터널을 작성하는 데 필요한 2단계(Phase) 매개변수를 정의하십시오.

Internet Key Exchange(IKE) 버전 2 오브젝트를 작성하십시오. IKEv2 제안 오브젝트에는
원격 액세스 및 사이트 대 사이트 VPN 정책을 정의할 때 IKEv2 제안을 작성하는 데 필요한
매개변수가 포함되어 있습니다. IKE는 IPsec 기반의 통신 관리를 용이하게 하는
관리 프로토콜입니다. 이 프로토콜은 IPsec 피어를 인증하는 데 사용되며
IPsec 암호화 키를 협상 및 분배하며 자동으로 IPsec 보안 연관(SA)을 설정합니다. 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <key value>
 ikev2 remote-authentication pre-shared-key <key value>
 ikev2 local-authentication pre-shared-key <key value>
```

IPsec 연결에 대한 IKEv2 정책 구성을 작성하십시오. IKEv2 정책 블록은
IKE 교환에 대한 매개변수를 설정합니다. 이 블록에서는 다음 매개변수가 설정됩니다.
* 암호화 알고리즘 - 이 예제의 경우에는 AES-256으로 설정됨
* 무결성 알고리즘 - 이 예제의 경우에는 SHA256으로 설정됨
* Diffie-Hellman 그룹 - IPsec는 Diffie-Hellman 알고리즘을 사용하여
피어 간의 초기 암호화 키를 생성합니다. 이 예제에서는 그룹 14로 설정됩니다.
* 의사 랜덤 기능(PRF) - IKEv2에는 IKEv2 터널 암호화에 필요한 암호화 자료 및 해시 오퍼레이션을 파생시키기 위한
알고리즘으로 사용할 별도의 방법이 필요합니다. 이 방법을 의사 랜덤 기능(pseudo-random function)이라고 하며 SHA로 설정됩니다.
* SA 수명 - 보안 연관의 수명을 설정합니다. 수명이 끝나면 재연결이 발생합니다. 36,000초로 설정하십시오.
* 작동 유형 - 기본값인 양방향으로 유지하십시오. ("실행 중 표시" 디스플레이에서는 명시적이지 않습니다.)

다음 코드 예제에서 보듯이 이 샘플 정책은 AES-256을 사용하여 보안 채널을 암호화합니다. 원격 피어의
ID 유효성을 검증하기 위해 SHA512 해시 알고리즘이 사용되고 키 생성을 위해 Diffie-Hellman 그룹 14가
사용됩니다. 그룹 14는 2048비트 암호화 블록을 사용합니다. 마지막으로
보안 연관에 대한 수명은 36,000초로 설정됩니다.

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* VPN에 대한 액세스 목록 및 암호화 맵 정의:

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## 로컬 IBM Cloud VPC와의 보안 연결 작성
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

보안 연결을 작성하기 위해 VPC 내에서 VPN 연결을 작성할 것이며 이는 2 VPC 예제와 유사합니다.

* `local_cidrs`를 VPC의 서브넷 값으로 설정하고 `peer_cidrs`를 Cisco ASAv의 서브넷 값으로 설정한 상태로 VPC 및 Cisco ASAv 사이의 VPN 연결과 함께 VPC 서브넷에 VPN 게이트웨이를 작성하십시오.

게이트웨이 상태는 VPN 게이트웨이가 작성 중인 동안 `pending`으로 표시되다가 작성이 완료되면 `available`로 변경됩니다. 작성에는 시간이 걸릴 수 있습니다. 
{:note}


![여기에 이미지 설명 입력](./images/vpc-vpn-asav-connection.png)

### 보안 연결의 상태 확인
{: #cisco-check-the-status-of-the-secure-connection}

IBM Cloud 콘솔을 통해 연결 상태를 확인할 수 있습니다. 또한 VSI를 사용하여 사이트에서 사이트로 `ping`을 시도할 수 있습니다.

![여기에 이미지 설명 입력](./images/vpc-vpn-asav-status.png)
