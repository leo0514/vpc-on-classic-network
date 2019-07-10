---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{: note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 원격 StrongSwan 피어와의 보안 연결 작성
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

이 문서는 Strongswan, 버전 Linux StrongSwan U5.3.5/K4.4.0-133-generic을 기반으로 합니다.

다음 예제 단계에서는 {{site.data.keyword.cloud}} API 또는 CLI를 사용하여 가상 사설 클라우드(VPC)를 작성하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [시작하기](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) 및 [API로 VPC 설정](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)을 참조하십시오.

## 예제 단계
{: #strongswan-example-steps}

원격 StrongSwan 피어에 대한 연결의 토폴로지는 두 VPC 사이에 VPN 연결을 작성하는 것과 유사합니다. 단, 연결의 한 측이 StrongSwan 장치로 대체됩니다.

![여기에 이미지 설명 입력](./images/vpc-vpn-sw-figure.png)

### 원격 StrongSwan 피어와의 보안 연결 작성 방법
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

**/etc**로 이동하여 이름이 **ipsec.abc.conf**와 유사한 새 사용자 정의 터널 구성 파일을 작성하십시오. 다음 행을 추가하여 **/etc/ipsec.conf**가 **ipsec.abc.conf**를 포함하도록 편집하십시오.

    include /etc/ipsec.abc.conf

VPN 피어가 원격 VPN 피어로부터 연결 요청을 수신하면 IPsec 1단계(Phase) 매개변수를 사용하여 보안 연결을 설정하고 해당 VPN 피어를 인증합니다. 그런 다음, 보안 정책이 연결을 허용하면 StrongSwan 장치가 IPsec 2단계(Phase) 매개변수를 사용하여 터널을 설정하고 IPsec 보안 정책을 적용합니다. 키 관리, 인증 및 보안 서비스가 IKE 프로토콜을 통해 동적으로 협상됩니다.

**해당 기능을 지원하려면 StrongSwan 장치에 의해 다음과 같은 일반 구성 단계가 수행되어야 합니다.**

* StrongSwan이 원격 피어를 인증하고 보안 연결을 설정하는 데 필요한 1단계(Phase) 매개변수를 정의하십시오.

* StrongSwan이 원격 피어를 사용하여 VPN 터널을 작성하는 데 필요한 2단계(Phase) 매개변수를 정의하십시오.
IBM Cloud VPC의 VPN 기능에 연결하기 위해 권장되는 구성은 다음과 같습니다.

1. 인증에서 `IKEv2` 선택
2. 1단계(Phase) 제안에서 `DH-group 2`를 사용하도록 설정
3. 1단계(Phase) 제안에서 `lifetime = 36000`을 설정
4. 2단계(Phase) 제안에서 PFS를 사용하지 않도록 설정
5. 2단계(Phase) 제안에서 `lifetime = 10800` 설정
6. 피어 및 서브넷의 정보를 2단계(Phase) 제안에 입력

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: screen}

`/etc/ipsec.secrets`에서 미리 공유한 키 설정

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

구성 파일 실행이 완료되면 StrongSwan 장치를 다시 시작하십시오.

```
 ipsec restart
```
{: screen}

### 로컬 IBM Cloud VPC와의 보안 연결 작성
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* 보안 연결을 작성하기 위해 VPC 내에서 VPN 연결을 작성할 것이며 이는 2 VPC 예제와 유사합니다.

* `local_cidrs`를 VPC의 서브넷 값으로 설정하고 `peer_cidrs`를 StrongSwan의 서브넷 값으로 설정한 상태로 VPC 및 StrongSwan 사이의 VPN 연결과 함께 VPC 서브넷에 VPN 게이트웨이를 작성하십시오.

게이트웨이 상태는 VPN 게이트웨이가 작성 중인 동안 `pending`으로 표시되다가 작성이 완료되면 `available`로 변경됩니다. 작성에는 시간이 걸릴 수 있습니다.
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### 보안 연결의 상태 확인
{: #strongswan-check-the-status-for-a-secure-connection}

IBM Cloud 콘솔을 통해 연결 상태를 확인할 수 있습니다. 또한 VSI를 사용하여 사이트에서 사이트로 `ping`을 시도할 수 있습니다.

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
