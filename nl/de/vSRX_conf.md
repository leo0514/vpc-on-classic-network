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


# Sichere Verbindung mit einem fernen Juniper vSRX-Peer erstellen
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

Dieses Dokument basiert auf Juniper vSRX, JUNOS Software Release [15.1X49-D123.3].

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen von VPCs unter Verwendung der {{site.data.keyword.cloud}}-API oder -CLI übersprungen. Weitere Informationen finden Sie in der [Einführung](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) und in [VPC-Einrichtung mit APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Beispielschritte
{: #vsrx-example-steps}

Die Topologie für die Verbindungsherstellung zum fernen Juniper vSRX-Peer hat Ähnlichkeit mit der [Erstellung einer VPN-Verbindung zwischen zwei VPCs](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc). Eine Seite der Verbindung wird jedoch durch die Juniper vSRX-Einheit ersetzt.

![Abbildungsbeschreibung](./images/vpc-vpn-vsrx-figure.png)

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit einen fernen Juniper vSRX-Peer
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

Wenn ein VPN-Peer eine Verbindungsanforderung von einem fernen VPN-Peer empfängt, verwendet sie IPsec-Parameter für Phase 1, um eine sichere Verbindung herzustellen und diesen VPN-Peer zu authentifizieren. Wenn die Sicherheitsrichtlinie die Verbindung zulässt, richtet die Juniper vSRX-Einheit dann unter Verwendung der IPsec-Parameter für Phase 2 den Tunnel ein und wendet die IPsec-Sicherheitsrichtlinie an. Schlüsselmanagement, Authentifizierung und Sicherheitsservices werden dynamisch über das IKE-Protokoll verhandelt.

**Damit diese Funktionen unterstützt werden, müssen die folgenden allgemeinen Konfigurationsschritte von der Juniper vSRX-Einheit ausgeführt werden:**

* Definieren der Parameter für Phase 1, die die Juniper vSRX-Einheit benötigt, um den fernen Peer zu authentifizieren und eine sichere Verbindung herzustellen.
* Definieren der Parameter von Phase 2, die die Juniper vSRX-Einheit benötigt, um einen VPN-Tunnel mit dem fernen Peer zu erstellen.
Um eine Verbindung mit der VPN-Funktionalität von IBM Cloud VPC herzustellen, wird die folgende Konfiguration empfohlen:

1. Wählen Sie in Phase 1 den Wert `IKEv1` aus.
2. Richten Sie den Richtlinienmodus ein, nicht den Weiterleitungsmodus.
3. Aktivieren Sie im Vorschlag für Phase 1 den Eintrag `DH-group 2`.
4. Legen Sie im Vorschlag für Phase 1 den Wert `lifetime = 36000` fest.
5. Aktivieren Sie im Vorschlag für Phase 2 den Wert 'PFS'.
6. Legen Sie im Vorschlag für Phase 2 den Wert `lifetime = 10800` fest.
7. Geben Sie beim Vorschlag für Phase 2 die Informationen für Ihren Peer und Ihr Teilnetz ein.
8. Lassen Sie UDP 500-Datenverkehr auf der externen Schnittstelle zu.

#### Bekannte Einschränkungen
{: #vsrx-known-limitations}

* Juniper vSRX unterstützt IKEv2 ausschließlich im _Richtlinienmodus_. Wenn Sie IKEv2 im Richtlinienmodus konfigurieren, wird daher der Fehler `IKEv2 requires bind-interface configuration as only route-based is supported` gemeldet. IBM Cloud VPC VPNaaS hingegen unterstützt derzeit nur den _Richtlinienmodus_. Daher müssen Sie IKEv1 in Phase 1 so konfigurieren, dass Juniper vSRX verwendet wird.

* Standardmäßig inaktiviert IBM Cloud VPC VPNaaS die PFS in Phase 2, wohingegen vSRX erfordert, dass PFS in Phase 2 _aktiviert_ ist. Aus diesem Grund müssen Sie eine neue IPsec-Richtlinie erstellen, die die Standardrichtlinie auf der VPC-VPNaaS-Seite ersetzt.

### Mit SSH bei Juniper vSRX anmelden, um die Konfiguration vorzunehmen
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### Das folgende Beispiel zeigt, wie Sie die Sicherheit einrichten:
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
                queue-size 2000; ## Warnung: 'queue-size' wird nicht weiter unterstützt
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

#### Das folgende Beispiel zeigt, wie Sie die Firewall einrichten:
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

Nachdem die Ausführung der Konfigurationsdatei abgeschlossen ist, sollten Sie den Verbindungsstatus über die Befehlszeilenschnittstelle (CLI) prüfen. Verwenden Sie dazu den folgenden Befehl:

```
 run show security ipsec security-associations
```
{: screen}

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit der lokalen IBM Cloud-VPC
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Zum Erstellen einer sicheren Verbindung erstellen Sie die VPN-Verbindung innerhalb Ihrer VPC, was dem Beispiel mit den 2 VPCs ähnelt.

Vergessen Sie nicht, dass Sie in Phase 2 PFS aktivieren müssen. Daher müssen Sie eine neue IPsec-Richtlinie erstellen, bevor Sie eine Verbindung erstellen.
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

Erstellen Sie ein VPN-Gateway auf Ihrem VPC-Teilnetz sowie eine VPN-Verbindung zwischen der VPC und der Juniper vSRX-Einheit. Legen Sie dabei für `local_cidrs` den Wert für das Teilnetz auf der VPC und für `peer_cidrs` den Wert für das Teilnetz auf der Juniper vSRX-Einheit fest.

Für das Gateway wird der Status 'Anstehend' (`pending`) angezeigt, solange das VPN-Gateway erstellt wird. Der Status wechselt zu 'Verfügbar' (`available`), sobald der Erstellungsvorgang abgeschlossen ist. Die Erstellung kann einige Zeit in Anspruch nehmen.
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### Status einer sicheren Verbindung prüfen
{: #vsrx-check-the-status-for-a-secure-connection}

Sie können den Status Ihrer Verbindung über die IBM Cloud-Konsole prüfen. Außerdem könnten Sie auch versuchen, von Site zu Site ein `Pingsignal` mit den VSIs abzusetzen.

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
