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


# 建立與遠端 Juniper vSRX 對等節點的安全連線
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

本文件是以 Juniper vSRX、JUNOS 軟體版本 [15.1X49-D123.3] 為基礎。

接下來的範例步驟會跳過使用 {{site.data.keyword.cloud}} API 或 CLI 來建立 Virtual Private Cloud 的必要步驟。如需相關資訊，請參閱[開始使用](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)及[使用 API 的 VPC 設定](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 範例步驟
{: #vsrx-example-steps}

連接至遠端 Juniper vSRX 對等節點的拓蹼類似於[在兩個 VPC 之間建立 VPN 連線](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)。不過，連線有一方會由 JunipervSRX 裝置取代。

![在這裡輸入影像說明](./images/vpc-vpn-vsrx-figure.png)

### 建立與遠端 Juniper vSRX 對等節點的安全連線
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

當 VPN 對等節點接收到來自遠端 VPN 對等節點的連線要求時，它會使用 IPsec「階段 1」參數來建立安全連線，並鑑別該 VPN 對等節點。然後，如果安全原則允許連線，Juniper vSRX 裝置會使用 IPsec「階段 2」參數來建立通道，並套用 IPsec 安全原則。金鑰管理、鑑別及安全服務會透過 IKE 通訊協定動態協議。

**若要支援這些功能，Juniper vSRX 裝置必須執行下列一般配置步驟：**

* 定義 Juniper vSRX 在鑑別遠端對等節點及建立安全連線時所需的「階段 1」參數。
* 定義 Juniper vSRX 在建立與遠端對等節點的 VPN 通道時所需的「階段 2」參數。若要連接至 IBM Cloud VPC 的 VPN 功能，建議您進行下列配置：

1. 選擇階段 1 中的 `IKEv1`
2. 設定原則模式，而非路徑模式；
3. 在「階段 1」提案中啟用 `DH-group 2`
4. 在「階段 1」提案中設定 `lifetime = 36000`
5. 在「階段 2」提案中啟用 PFS
6. 在「階段 2」提案中設定 `lifetime = 10800`
7. 在「階段 2」提案中輸入對等節點及子網路的資訊
8. 容許外部介面上具有 UDP 500 資料流量

#### 已知限制
{: #vsrx-known-limitations}

* Juniper vSRX 僅在_路徑模式_ 下支援 IKEv2。因此，如果您在原則模式中設定 IKEv2，則會看到此錯誤：`IKEv2 需要連結介面配置，因為僅支援路徑型`。不過，IBM Cloud VPC VPNaaS 目前僅支援_原則模式_，因此您必須在「階段 1」設定 IKEv1 以使用 Juniper vSRX。

* 依預設，IBM Cloud VPC VPNaaS 會停用「階段 2」中的 PFS，而 vSRX 需要在「階段 2」中將 PFS 設為_已啟用_。因此，您必須建立新的 IPsec 原則，以取代 VPC VPNaaS 端的預設原則。

### 登入 Juniper vSRX 以使用 SSH 進行配置
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### 以下是如何設定安全的範例：
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

#### 以下是如何設定防火牆的範例：
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

在配置檔完成執行之後，您可以使用下列指令，從 CLI 檢查連線狀態：

```
 run show security ipsec security-associations
```
{: screen}

### 建立與本端 IBM Cloud VPC 的安全連線
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

若要建立安全連線，您要在 VPC 內建立 VPN 連線，其類似於 2 VPC 範例。

請記住，您必須在「階段 2」中啟用 PFS，因此，在建立連線之前，您必須先建立新的 IPsec 原則。
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

在 VPC 子網路上建立 VPN 閘道，以及在 VPC 與 Juniper vSRX 之間建立 VPN 連線，將 `local_cidrs` 設為 VPC 上的子網路值，並將 `peer_cidrs` 設為 Juniper vSRX 上的子網路值。

建立 VPN 閘道時，閘道狀態會顯示為 `pending`，建立完成之後，狀態會變成 `available`。建立作業可能需要一些時間。
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### 檢查安全連線的狀態
{: #vsrx-check-the-status-for-a-secure-connection}

您可以透過 IBM Cloud 主控台檢查連線狀態。您也可以嘗試使用 VSI 執行點對點 `ping`。

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
