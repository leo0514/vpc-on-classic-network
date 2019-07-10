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


# リモート Juniper vSRX ピアとのセキュア接続の作成
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

この資料は、Juniper vSRX、JUNOS Software Release [15.1X49-D123.3] に基づいています。

以下の手順の例では、{{site.data.keyword.cloud}} API または CLI を使用して仮想プライベート・クラウドを作成するという前提条件となる手順を省略しています。 詳しくは、[概要](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)および [API を使用した VPC のセットアップ](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)を参照してください。

## 手順の例
{: #vsrx-example-steps}

リモートの Juniper vSRX ピアに接続するためのトポロジーは、[2 つの VPC 間に VPN 接続を作成](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)するのと似ています。 ただし、接続の一方の側が Juniper vSRX ユニットに置き換わっている点が異なります。

![画像の説明をここに入力](./images/vpc-vpn-vsrx-figure.png)

### リモート Juniper vSRX ピアとのセキュア接続を作成するには、以下のようにします
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

VPN ピアは、リモート VPN ピアから接続要求を受け取ると、IPsec フェーズ 1 のパラメーターを使用してセキュア接続を確立し、その VPN ピアを認証します。 その後、セキュリティー・ポリシーにより接続が許可されると、Juniper vSRX ユニットは IPsec フェーズ 2 のパラメーターを使用してトンネルを確立し、IPsec セキュリティー・ポリシーを適用します。 鍵管理サービス、認証サービス、およびセキュリティー・サービスは、IKE プロトコルを使用して動的にネゴシエーションされます。

**これらの機能をサポートするために、以下の一般的な構成ステップを Juniper vSRX ユニットで実行する必要があります。**

* フェーズ 1 のパラメーターを定義します。これは、Juniper vSRX がリモート・ピアの認証とセキュア接続の確立を行うために必要です。
* フェーズ 2 のパラメーターを定義します。これは、Juniper vSRX がリモート・ピアへの VPN トンネルを作成するために必要です。
IBM Cloud VPC の VPN 機能に接続するには、以下の構成が推奨されます。

1. フェーズ 1 で `IKEv1` を選択します
2. 経路モードではなくポリシー・モードをセットアップします
3. フェーズ 1 プロポーザルで `DH-group 2` を有効にします
4. フェーズ 1 プロポーザルで `lifetime = 36000` を設定します
5. フェーズ 2 プロポーザルで PFS を有効にします
6. フェーズ 2 プロポーザルで `lifetime = 10800` を設定します
7. フェーズ 2 プロポーザルでピアとサブネットの情報を入力します
8. 外部インターフェースで UDP 500 トラフィックを許可します

#### 既知の制限事項
{: #vsrx-known-limitations}

* Juniper vSRX は_経路モード_ でのみ IKEv2 をサポートしています。 したがって、ポリシー・モードで IKEv2 をセットアップすると、エラー `IKEv2 requires bind-interface configuration as only route-based is supported` が表示されます。 ただし、IBM Cloud VPC の VPNaaS は現在_ポリシー・モード_ のみをサポートしているため、Juniper vSRX を使用するには、フェーズ 1 で IKEv1 をセットアップする必要があります。

* デフォルトでは、IBM Cloud VPC の VPNaaS はフェーズ 2 で PFS を無効にしますが、vSRX はフェーズ 2 で PFS を_有効_ にする必要があります。このため、VPC の VPNaaS 側でデフォルト・ポリシーを置き換える新しい IPsec ポリシーを作成する必要があります。

### Juniper vSRX にログインし、SSH を使用して構成します
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### セキュリティーのセットアップ方法の例を以下に示します。
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

#### ファイアウォールのセットアップ方法の例を以下に示します。
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

構成ファイルの実行が完了したら、CLI から以下のコマンドを使用して接続状況を確認できます。

```
 run show security ipsec security-associations
```
{: screen}

### ローカル IBM Cloud VPC とのセキュア接続を作成するには、以下のようにします
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

セキュア接続を作成するには、VPC 内で VPN 接続を作成します。これは、2 つの VPC の例に似ています。

フェーズ 2 で PFS を有効にする必要があるため、接続を作成する前に新しい IPsec ポリシーを作成する必要があることに注意してください。
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

VPC サブネットに VPN ゲートウェイを作成し、VPC と Juniper vSRX の間に VPN 接続を作成し、`local_cidrs` を VPC のサブネット値に設定し、`peer_cidrs` を Juniper vSRX のサブネット値に設定します。

VPN ゲートウェイの作成中はゲートウェイの状況が `pending` と表示されます。作成が完了すると状況が `available` になります。 作成にはしばらく時間がかかる場合があります。
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### セキュア接続の状況の確認
{: #vsrx-check-the-status-for-a-secure-connection}

IBM Cloud コンソールから接続の状況を確認できます。 また、VSI を使用してサイト間で `ping` を実行することもできます。

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
