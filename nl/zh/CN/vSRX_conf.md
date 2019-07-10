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


# 创建与远程 Juniper vSRX 同级的安全连接
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

本文档基于 Juniper vSRX JUNOS 软件发行版 [15.1X49-D123.3]。

以下示例步骤跳过了使用 {{site.data.keyword.cloud}} API 或 CLI 来创建虚拟私有云的先决条件步骤。有关更多信息，请参阅[入门](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)和[使用 API 设置 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)。

## 示例步骤
{: #vsrx-example-steps}

用于连接到远程 Juniper vSRX 同级的拓扑类似于[在两个 VPC 之间创建 VPN 连接](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)。但是，连接的一侧会替换为 Juniper vSRX 单元。

![在此输入图像描述](./images/vpc-vpn-vsrx-figure.png)

### 创建与远程 Juniper vSRX 同级的安全连接
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

VPN 同级收到来自远程 VPN 同级的连接请求时，会使用 IPsec 第 1 阶段参数来建立安全连接，并对该 VPN 同级进行认证。随后，如果安全策略允许连接，Juniper vSRX 单元将使用 IPsec 第 2 阶段参数来建立隧道，并应用 IPsec 安全策略。密钥管理、认证和安全服务均通过 IKE 协议动态协商。

**为了支持这些功能，Juniper vSRX 单元必须执行以下常规配置步骤：**

* 定义 Juniper vSRX 对远程同级进行认证并建立安全连接所需的第 1 阶段参数。
* 定义 Juniper vSRX 创建与远程同级的 VPN 隧道所需的第 2 阶段参数。要连接到 IBM Cloud VPC 的 VPN 功能，建议使用以下配置：

1. 在第 1 阶段中选择 `IKEv1`；
2. 设置策略方式，而不是路由方式；
3. 在第 1 阶段建议中启用 `DH-group 2`
4. 在第 1 阶段建议中设置 `lifetime = 36000`
5. 在第 2 阶段建议中启用 PFS
6. 在第 2 阶段建议中设置 `lifetime = 10800`
7. 在第 2 阶段建议中输入同级和子网的信息
8. 允许外部接口上的 UDP 500 流量。

#### 已知限制
{: #vsrx-known-limitations}

* Juniper vSRX 仅在_路由方式_下支持 IKEv2。因此，如果将 IKEv2 设置为策略方式，那么将看到错误 `IKEv2 需要绑定接口配置，因为仅支持基于路由的方式`。但是，IBM Cloud VPC VPNaaS 目前仅支持_策略方式_，因此必须在第 1 阶段中设置 IKEv1 才能使用 Juniper vSRX。

* 缺省情况下，IBM Cloud VPC VPNaaS 在第 2 阶段中禁用了 PFS，但 vSRX 需要 PFS 在第 2 阶段中处于_已启用_状态。因此，必须创建新的 IPsec 策略来替换 VPC VPNaaS 侧的缺省策略。

### 使用 SSH 登录到 Juniper vSRX 以对其进行配置
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### 下面是如何设置安全性的示例：
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

#### 下面是如何设置防火墙的示例：
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

在配置文件执行完毕后，可以通过 CLI 使用以下命令来检查连接状态：

```
 run show security ipsec security-associations
```
{: screen}

### 创建与本地 IBM Cloud VPC 的安全连接
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

要创建安全连接，需要在 VPC 中创建 VPN 连接，这类似于 2 个 VPC 的示例。

请记住，必须在第 2 阶段中启用 PFS，因此必须在创建连接之前，先创建新的 IPsec 策略。
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

在 VPC 子网上创建 VPN 网关，并创建 VPC 与 Juniper vSRX 之间的 VPN 连接，将 `local_cidrs` 设置为 VPC 上的子网值，将 `peer_cidrs` 设置为 Juniper vSRX 上的子网值。

创建 VPN 网关时，网关状态显示为 `pending`，创建完成后，状态会变为 `available`。创建过程可能需要一些时间。
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### 检查安全连接的状态
{: #vsrx-check-the-status-for-a-secure-connection}

可以通过 IBM Cloud 控制台来检查连接的状态。此外，还可以尝试使用 VSI 执行站点到站点的 `ping` 操作。

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
