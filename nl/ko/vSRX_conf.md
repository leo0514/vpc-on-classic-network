---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Juniper, vSRX, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:important: .important}
{:table: .aria-labeledby="caption"}
{:download: .download}
{:note: .note}
{:DomainName: data-hd-keyref="DomainName"}


# 원격 Juniper vSRX 피어와의 보안 연결 작성
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

이 문서는 Juniper vSRX, JUNOS Software 릴리스 [15.1X49-D123.3]을 기반으로 합니다.

다음 예제 단계에서는 {{site.data.keyword.cloud}} API 또는 CLI를 사용하여 가상 사설 클라우드(VPC)를 작성하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [시작하기](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) 및 [API로 VPC 설정](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)을 참조하십시오.

## 예제 단계
{: #vsrx-example-steps}

원격 Juniper vSRX 피어에 대한 연결의 토폴로지는 [두 VPC 사이에 VPN 연결을 작성](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)하는 것과 유사합니다. 단, 연결의 한 측이 Juniper vSRX 장치로 대체됩니다.

![여기에 이미지 설명 입력](./images/vpc-vpn-vsrx-figure.png)

### 원격 Juniper vSRX 피어와의 보안 연결 작성 방법
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

VPN 피어가 원격 VPN 피어로부터 연결 요청을 수신하면 IPsec 1단계(Phase) 매개변수를 사용하여 보안 연결을 설정하고 해당 VPN 피어를 인증합니다. 그런 다음, 보안 정책이 연결을 허용하면 Juniper vSRX 장치가 IPsec 2단계(Phase) 매개변수를 사용하여 터널을 설정하고 IPsec 보안 정책을 적용합니다. 키 관리, 인증 및 보안 서비스가 IKE 프로토콜을 통해 동적으로 협상됩니다.

**해당 기능을 지원하려면 Juniper vSRX 장치에 의해 다음과 같은 일반 구성 단계가 수행되어야 합니다.**

* Juniper vSRX가 원격 피어를 인증하고 보안 연결을 설정하는 데 필요한 1단계(Phase) 매개변수를 정의하십시오.
* Juniper vSRX가 원격 피어를 사용하여 VPN 터널을 작성하는 데 필요한 2단계(Phase) 매개변수를 정의하십시오.
IBM Cloud VPC의 VPN 기능에 연결하기 위해 권장되는 구성은 다음과 같습니다.

1. 1단계(Phase)에서 `IKEv1` 선택
2. 라우트 모드가 아니라 정책 모드 설정
3. 1단계(Phase) 제안에서 `DH-group 2`를 사용하도록 설정
4. 1단계(Phase) 제안에서 `lifetime = 36000`을 설정
5. 2단계(Phase) 제안에서 PFS를 사용하도록 설정
6. 2단계(Phase) 제안에서 `lifetime = 10800` 설정
7. 피어 및 서브넷의 정보를 2단계(Phase) 제안에 입력
8. 외부 인터페이스에서 UDP 500 트래픽 허용

#### 알려진 제한사항
{: #vsrx-known-limitations}

* Juniper vSRX는 _라우트 모드_에서만 IKEv2를 지원합니다. 따라서 IKEv2를 정책 모드로 설정하면 `IKEv2 requires bind-interface configuration as only route-based is supported` 오류가 표시됩니다. 그러나 IBM Cloud VPC VPNaaS는 현재 _정책 모드_만 지원합니다. 따라서 Juniper vSRX를 사용하려면 1단계(Phase)에서 IKEv1을 설정해야 합니다.

* 기본적으로 IBM Cloud VPC VPNaaS는 2단계(Phase)에서 PFS를 사용 안함으로 설정하며 vSRX에서는 2단계(Phase)에서 PFS가 _사용됨_으로 설정되어야 합니다. 이런 이유로 VPC VPNaaS 측에서 기본 정책을 대체할 새 IPsec 정책을 작성해야 합니다.

### Juniper vSRX에 로그인하여 SSH를 사용하여 구성
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### 다음은 보안을 설정하는 예제입니다.
{: #vsrx-here-s-an-example-of-how-to-set-up-security}

```

admin@Juniper-vSRX# show security    

log {
    mode stream;
    report;
}
ike {
    traceoptions {
        file ike_log_20 size 10240000;
        flag all;
    }
    proposal ike-proposal-1 {
        authentication-method pre-shared-keys;
        dh-group group2;
        authentication-algorithm sha-256;
        encryption-algorithm aes-256-cbc;
    }
    policy ike1 {
        mode main;
        proposals ike-proposal-1;
        pre-shared-key ascii-text "$9$sO2JGjHqfQFiH0BRhrl"; ## SECRET-DATA
    }
    gateway gw1 {
        ike-policy ike1;
        address 169.45.74.119;
        dead-peer-detection always-send;
        no-nat-traversal;
        local-identity inet 169.61.195.195;
        external-interface ge-0/0/1.0;
        local-address 169.61.195.195;
        version v1-only;
    }
}
ipsec {
    proposal AES128cbc-SHA256-esp {
        protocol esp;
        authentication-algorithm hmac-sha-256-128;
        encryption-algorithm aes-128-cbc;
    }
    policy ipsec1 {
        perfect-forward-secrecy {
            keys group2;
        }
        proposals AES128cbc-SHA256-esp;
    }
    vpn to-strongswan {
        ike {
            gateway gw1;
            proxy-identity {
                local 10.93.152.152/29;
                remote 10.160.26.64/26;
                service any;
            }
            ipsec-policy ipsec1;
        }
        establish-tunnels immediately;
    }
}
address-book {
    global {
        address SL_PRIV_MGMT 10.93.160.12/32;
        address SL_PUB_MGMT 169.61.195.195/32;
        }
    }
    local_cidr {
        address local_cidr 10.93.160.0/26;
        attach {
            zone SL-PRIVATE;
        }
    }
    remote_cidr {
        address remote 10.160.26.64/26;
        attach {
            zone SL-PUBLIC;
        }
    }
}
screen {
    ids-option untrust-screen {
        icmp {
            ping-death;
        }
        ip {
            source-route-option;
            tear-drop;
        }
        tcp {
            syn-flood {
                alarm-threshold 1024;
                attack-threshold 200;
                source-threshold 1024;
                destination-threshold 2048;
                queue-size 2000; ## Warning: 'queue-size' is deprecated
                timeout 20;
            }
            land;
        }
    }
}
policies {
    from-zone SL-PRIVATE to-zone SL-PRIVATE {
        policy Allow_Management {
            match {
                source-address any;
                destination-address any;
                application any;
            }
            then {
                permit;
            }
        }
    }
    from-zone SL-PUBLIC to-zone SL-PUBLIC {
        policy pub2pub {
            match {
                source-address any;
                destination-address SL_PUB_MGMT;
                application any;
            }
            then {
                permit;
            }
        }
        policy Allow_Management {
            match {
                source-address any;
                destination-address SL_PUB_MGMT;
                application [ junos-ssh junos-https junos-http junos-icmp-ping ];
            }
            then {
                permit;
            }
        }
    }
    from-zone SL-PRIVATE to-zone SL-PUBLIC {
        policy out {
            match {
                source-address any;
                destination-address any;
                application any;
            }
            then {
                permit {
                    tunnel {
                        ipsec-vpn to-strongswan;
                    }
                }
            }
        }
    }
    from-zone SL-PUBLIC to-zone SL-PRIVATE {
        policy in {
            match {
                source-address any;
                destination-address any;
                application any;
            }
            then {
                permit {
                    tunnel {
                        ipsec-vpn to-strongswan;
                    }
                }
            }
        }
    }
}
zones {
    security-zone SL-PRIVATE {
        interfaces {
            ge-0/0/0.0 {
                host-inbound-traffic {
                    system-services {
                        all;
                    }
                }
            }
            st0.1;
            ge-0/0/0.986 {
                host-inbound-traffic {
                    system-services {
                        all;
                    }
                }
            }
        }
    }
    security-zone SL-PUBLIC {
        interfaces {
            ge-0/0/1.0 {
                host-inbound-traffic {
                    system-services {
                        all;
                    }
                }
            }
        }
    }
}

[edit]

```
{: screen}

#### 다음은 방화벽을 설정하는 예제입니다.
{: #vsrx-here-s-an-example-of-how-to-set-up-the-firewall}

```
admin@Juniper-vSRX# show firewall filter PROTECT-IN term VPN_IKE
from {
    destination-address {
        169.61.195.195/32;
        10.93.160.12/32;
    }
    protocol udp;
    port 500;
}
then accept;

[edit]
```
{: screen}

구성 파일 실행이 완료되면 다음 명령을 사용하여 CLI에서 연결 상태를 확인할 수 있습니다.

```
 run show security ipsec security-associations
```
{: screen}

### 로컬 IBM Cloud VPC와의 보안 연결 작성
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

보안 연결을 작성하기 위해 VPC 내에서 VPN 연결을 작성할 것이며 이는 2 VPC 예제와 유사합니다.

2단계(Phase)에서 PFS를 사용으로 설정해야 합니다. 따라서 연결을 작성하기 전에 새 IPsec 정책을 작성해야 합니다.
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

`local_cidrs`를 VPC의 서브넷 값으로 설정하고 `peer_cidrs`를 Juniper vSRX의 서브넷 값으로 설정한 상태로 VPC 및 Juniper vSRX 사이의 VPN 연결과 함께 VPC 서브넷에 VPN 게이트웨이를 작성하십시오.

게이트웨이 상태는 VPN 게이트웨이가 작성 중인 동안 `pending`으로 표시되다가 작성이 완료되면 `available`로 변경됩니다. 작성에는 시간이 걸릴 수 있습니다.
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### 보안 연결의 상태 확인
{: #vsrx-check-the-status-for-a-secure-connection}

IBM Cloud 콘솔을 통해 연결 상태를 확인할 수 있습니다. 또한 VSI를 사용하여 사이트에서 사이트로 `ping`을 시도할 수 있습니다.

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
