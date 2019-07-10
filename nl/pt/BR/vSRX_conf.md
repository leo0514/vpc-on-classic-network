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


# Criando uma conexão segura com um peer Juniper vSRX remoto
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

Este documento é baseado no Juniper vSRX, Liberação de Software JUNOS [15.1X49-D123.3].

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da API ou da CLI do {{site.data.keyword.cloud}} para criar Nuvens Particulares Virtuais. Para obter mais informações, consulte [Introdução](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configuração de VPC com APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Etapas de exemplo
{: #vsrx-example-steps}

A topologia para conexão com o peer do Juniper vSRX remoto é semelhante a [criar uma conexão VPN entre duas VPCs](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc). No entanto, um lado da conexão é substituído pela unidade Juniper vSRX.

![inserir descrição de imagem aqui](./images/vpc-vpn-vsrx-figure.png)

### Para criar uma conexão segura com um peer Juniper vSRX remoto
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

Quando um peer do VPN recebe uma solicitação de conexão de um peer do VPN remoto, ele usa os parâmetros da Fase 1 do IPsec para estabelecer uma conexão segura e autenticar esse peer do VPN. Em seguida, se a política de segurança permitir a conexão, a unidade Juniper vSRX estabelecerá o túnel usando os parâmetros da Fase 2 do IPsec e aplicará a política de segurança do IPsec. Os serviços de gerenciamento de chaves, de autenticação e de segurança são negociados dinamicamente por meio do protocolo IKE.

**Para suportar essas funções, as etapas de configuração gerais a seguir devem ser executadas pela unidade Juniper vSRX:**

* Defina os parâmetros da Fase 1 que o Juniper vSRX requer para autenticar o peer remoto e estabelecer uma conexão segura.
* Defina os parâmetros da Fase 2 que o Juniper vSRX requer para criar um túnel VPN com o peer remoto.
Para se conectar ao recurso VPN do IBM Cloud VPC, recomendamos a configuração a seguir:

1. Escolha `IKEv1` na fase 1;
2. Configure o modo de política, não o modo de rota;
3. Ative `DH-group 2` na proposta da Fase 1
4. Configure `lifetime = 36000` na proposta da Fase 1
5. Ative o PFS na proposta da Fase 2
6. Configure `lifetime = 10800` na proposta da Fase 2
7. Insira as informações de seu peer e sub-rede na proposta da Fase 2
8. Permita tráfego UDP 500 na interface externa.

#### Limitações Conhecidas
{: #vsrx-known-limitations}

* O Juniper vSRX suporta IKEv2 no _modo de rota_ apenas. Portanto, se você configurar o IKEv2 no modo de política, verá o erro `IKEv2 requires bind-interface configuration as only route-based is supported`. No entanto, o VPNaaS do IBM Cloud VPC atualmente suporta apenas o _modo de política_, portanto, deve-se configurar o IKEv1 na Fase 1 para usar o Juniper vSRX.

* Por padrão, o VPNaaS do IBM Cloud VPC desativa o PFS na Fase 2 e o vSRX requer que o PFS seja _ativado_ na Fase 2. Por essa razão, deve-se criar uma nova política de IPsec para substituir a política padrão no lado VPNaaS da VPC.

### Efetue login no Juniper vSRX para configurá-lo usando SSH
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### Aqui está um exemplo de como configurar a segurança:
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
                source-address any; destination-address any; application any; } then {
                permit; }
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
                permit; }
        }
        policy Allow_Management {
            match {
                source-address any;     
                destination-address SL_PUB_MGMT;
                application [ junos-ssh junos-https junos-http junos-icmp-ping ];
            }
            then {
                permit; }
        }
    }
    from-zone SL-PRIVATE to-zone SL-PUBLIC {
        policy out {
            match {
                source-address any; destination-address any; application any; } then {
                permit {
                    tunnel {
                        ipsec-vpn to-strongswan; }
                }
            }
        }
    }
    from-zone SL-PUBLIC to-zone SL-PRIVATE {
        policy in {
            match {
                source-address any; destination-address any; application any; } then {
                permit {
                    tunnel {
                        ipsec-vpn to-strongswan; }
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
                        all; }
                }
            }
            st0.1;
            ge-0/0/0.986 {
                host-inbound-traffic {
                    system-services {
                        all; }
                }
            }
        }
    }
    security-zone SL-PUBLIC {
        interfaces {
            ge-0/0/1.0 {
                host-inbound-traffic {
                    system-services {
                        all; }
                }
            }
        }
    }
}

[edit]

```
{: screen}

#### Aqui está um exemplo de como configurar o firewall:
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

Depois que o arquivo de configuração tiver concluído a execução, você poderá verificar o status da conexão por meio da CLI, com o comando a seguir:

```
 run show security ipsec security-associations
```
{: screen}

### Para criar uma conexão segura com o IBM Cloud VPC local
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Para criar uma conexão segura, você criará a conexão VPN dentro de seu VPC, que é semelhante ao exemplo 2 de VPC.

Lembre-se de que você deve ativar o PFS na Fase 2, portanto, deve-se criar uma nova política IPsec antes de criar uma conexão.
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

Crie um gateway VPN em sua sub-rede da VPC juntamente com uma conexão VPN entre a VPC e o Juniper vSRX, configurando `local_cidrs` para o valor de sub-rede na VPC e `peer_cidrs` para o valor de sub-rede no Juniper vSRX.

O status do gateway aparece como `pending` enquanto o gateway VPN está sendo criado e o status se torna `available` assim que a criação é concluída. A criação pode levar algum tempo.
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### Verifique o status para uma conexão segura
{: #vsrx-check-the-status-for-a-secure-connection}

É possível verificar o status de sua conexão por meio do console do IBM Cloud. Além disso, é possível tentar executar um `ping` de site para site usando os VSIs.

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
