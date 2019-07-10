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


# Creazione di una connessione sicura con un peer Juniper vSRX remoto
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

Questo documento si basa su Juniper vSRX, JUNOS Software Release [15.1X49-D123.3].

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API {{site.data.keyword.cloud}} per creare i VPC (Virtual Private Cloud). Per ulteriori informazioni, consulta [Introduzione](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configurazione di VPC con le API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Istruzioni di esempio
{: #vsrx-example-steps}

La topologia per la connessione al peer Juniper vSRX remoto è simile alla [creazione di una connessione VPN tra due VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc). Tuttavia, un lato della connessione viene sostituito dall'unità Juniper vSRX.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-vsrx-figure.png)

### Per creare una connessione sicura con un peer Juniper vSRX remoto
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

Quando un peer VPN riceve una richiesta di connessione da un peer VPN remoto, utilizza i parametri IPsec Phase 1 per stabilire una connessione sicura e autenticare tale peer VPN. Successivamente, se la politica di sicurezza consente la connessione, l'unità Juniper vSRX stabilisce il tunnel utilizzando i parametri IPsec Phase 2 e applica la politica di sicurezza IPsec. I servizi di sicurezza, autenticazione e di gestione della chiave vengono negoziati in modo dinamico tramite il protocollo IKE.

**Per supportare queste funzioni, devono essere eseguiti i seguenti passi di configurazione generale dall'unità Juniper vSRX:**

* Definisci i parametri Phase 1 necessari a Juniper vSRX per autenticare il peer remoto e stabilire una connessione sicura.
* Definisci i parametri Phase 2 necessari a Juniper vSRX per creare un tunnel VPN con il peer remoto.
Per la connessione alla funzionalità VPN di IBM Cloud VPC, consigliamo la seguente configurazione:

1. Scegli `IKEv1` in Phase 1;
2. Configura la modalità della politica, non la modalità di instradamento;
3. Abilita `DH-group 2` nella proposta Phase 1
4. Imposta `lifetime = 36000` nella proposta Phase 1
5. Abilita PFS nella proposta Phase 2
6. Imposta `lifetime = 10800` nella proposta Phase 2
7. Immetti le informazioni del tuo peer e della tua sottorete nella proposta Phase 2
8. Consenti il traffico UDP 500 sull'interfaccia esterna.

#### Limitazioni note
{: #vsrx-known-limitations}

* Juniper vSRX supporta IKEv2 solo nella _modalità di instradamento_. Pertanto, se configuri IKEv2 nella modalità della politica, visualizzerai l'errore `IKEv2 requires bind-interface configuration as only route-based is supported`. Tuttavia, IBM Cloud VPC VPNaaS al momento supporta solo la _modalità della politica_, per cui devi configurare IKEv1 nella Phase 1 in modo che utilizzi Juniper vSRX.

* Per impostazione predefinita, IBM Cloud VPC VPNaaS disabilita PFS nella Phase 2 mentre vSRX necessita che PFS sia _abilitato_ nella Phase 2. Per questo motivo, devi creare una nuova politica IPsec per sostituire la politica predefinita sul lato VPC VPNaaS.

### Accedi a Juniper vSRX per configurarlo utilizzando SSH
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### Ecco un esempio di come configurare la sicurezza:
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

#### Ecco un esempio di come configurare il firewall:
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

Dopo che il file di configurazione ha terminato l'esecuzione, puoi controllare lo stato della connessione dalla CLI, con il seguente comando:

```
 run show security ipsec security-associations
```
{: screen}

### Per creare una connessione sicura con l'IBM Cloud VPC locale
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Per creare una connessione sicura, creerai la connessione VPN all'interno del tuo VPC, che è simile all'esempio dei 2 VPC.

Ricorda: devi abilitare PFS nella Phase 2, per cui devi creare una nuova politica IPsec prima di creare una connessione.
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

Crea un gateway VPN sulla tua sottorete VPC insieme a una connessione VPN tra il VPC e Juniper vSRX, impostando `local_cidrs` come valore della sottorete nel VPC e `peer_cidrs` come valore della sottorete nel Juniper vSRX.

Lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto.
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### Controlla lo stato per una connessione sicura
{: #vsrx-check-the-status-for-a-secure-connection}

Puoi controllare lo stato della tua connessione tramite la console IBM Cloud. Inoltre, puoi provare ad eseguire il `ping` tra i siti utilizzando le VSI.

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
