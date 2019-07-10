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


# Création d'une connexion sécurisée avec un homologue Juniper vSRX distant
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

Ce document est basé sur Juniper vSRX, JUNOS Software Release [15.1X49-D123.3].

Les exemples qui suivent ignorent les étapes prérequises de l'utilisation de l'API ou de l'interface de ligne de commande {{site.data.keyword.cloud}} pour créer des clouds privés virtuels. Pour plus d'informations, voir [Initiation](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) et [Configuration de VPC avec des API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Exemple d'étapes
{: #vsrx-example-steps}

La topologie de connexion à l'homologue Juniper vSRX distant est similaire à [la création d'une connexion VPN entre deux VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc). Cependant, l'un des côtés de la connexion est remplacé par l'unité Juniper vSRX.

![entrer la description de l'image ici](./images/vpc-vpn-vsrx-figure.png)

### Pour créer une connexion sécurisée avec un homologue Juniper vSRX distant
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

Lorsqu'un homologue VPN reçoit une demande de connexion d'un homologue VPN distant, il utilise les paramètres IPsec Phase 1 pour établir une connexion sécurisée et authentifier cet homologue VPN. Ensuite, si la stratégie de sécurité autorise la connexion, l'unité Juniper vSRX établit le tunnel à l'aide des paramètres IPsec Phase 2 et applique la stratégie de sécurité IPsec. Les services de gestion de clés, d'authentification et de sécurité sont négociés de manière dynamique via le protocole IKE.

**Pour prendre en charge ces fonctions, les étapes de configuration générales suivantes doivent être effectuées par l'unité Juniper vSRX :**

* Définissez les paramètres de phase 1 requis par l'unité Juniper vSRX pour authentifier l'homologue distant et établir une connexion sécurisée.
* Définissez les paramètres de phase 2 nécessaires à l'unité Juniper vSRX pour créer un tunnel VPN avec l'homologue distant.
Pour vous connecter à la fonctionnalité VPN d'IBM Cloud VPC, nous vous recommandons la configuration suivante :

1. Sélectionnez `IKEv1` en phase 1
2. Définissez le mode de stratégie, et non le mode route ;
3. Activez `DH-group 2` dans la proposition de phase 1
4. Définissez `lifetime = 36000` dans la proposition de phase 1
5. Activez PFS dans la proposition de phase 2
6. Définissez `lifetime = 10800` dans la proposition de phase 2
7. Saisissez les informations de l'homologue et du sous-réseau dans la proposition de phase 2
8. Autoriser le trafic UDP 500 sur l'interface externe.

#### Limitations connues
{: #vsrx-known-limitations}

* Juniper vSRX prend en charge IKEv2 en _mode route_ uniquement. Par conséquent, si vous configurez IKEv2 en mode stratégie, le message d'erreur suivant s'affiche : `IKEv2 requires bind-interface configuration as only route-based is supported`. Toutefois, IBM Cloud VPC VPNaaS ne prend actuellement en charge que le _mode stratégie_. Vous devez donc configurer IKEv1 en phase 1 pour utiliser Juniper vSRX.

* Par défaut, IBM Cloud VPC VPNaaS désactive PFS en phase 2 et vSRX exige que PFS soit _activé_ en phase 2. Pour cette raison, vous devez créer une nouvelle stratégie IPsec pour remplacer la stratégie par défaut du côté VPC VPNaaS.

### Connectez-vous à Juniper vSRX pour le configurer à l'aide de SSH
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### Voici un exemple de configuration de la sécurité :
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

#### Voici un exemple de configuration du pare-feu :
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

Une fois l'exécution du fichier de configuration terminée, vous pouvez vérifier le statut de la connexion à partir de la CLI à l'aide de la commande suivante :

```
 run show security ipsec security-associations
```
{: screen}

### Pour créer une connexion sécurisée avec l'instance locale d'IBM Cloud VPC
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Pour créer une connexion sécurisée, créez une connexion VPN au sein de votre VPC, similaire à l'exemple VPC 2.

N'oubliez pas que vous devez activer PFS au cours de la phase 2 et que vous devez donc créer une nouvelle stratégie IPSec avant de créer une connexion.
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

Créez une passerelle VPN sur votre sous-réseau VPC, ainsi qu'une connexion VPN entre le VPC et l'instance Juniper vSRX, en définissant `local_cidrs` sur la valeur de sous-réseau sur le VPC et `peer_cidrs` sur la valeur de sous-réseau sur Juniper vSRX.

REMARQUE : Le statut de la passerelle apparaît en tant qu'`en attente` lors de la création de la passerelle VPN et devient `disponible` une fois la création terminée. La création peut prendre du temps.
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### Vérification du statut d'une connexion sécurisée
{: #vsrx-check-the-status-for-a-secure-connection}

Vous pouvez vérifier le statut de votre connexion via la console IBM Cloud. En outre, vous pouvez tenter de créer un fichier `ping` d'un site à l'autre à l'aide des VSI.

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
