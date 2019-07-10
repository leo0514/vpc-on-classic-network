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


# Creación de una conexión segura con un igual de Juniper vSRX remoto
{: #creating-a-secure-connection-with-a-remote-juniper-vsrx-peer}

Este documento se basa en Juniper vSRX, el release de software de JUNOS [15.1X49-D123.3].

Los pasos de ejemplo que siguen omiten los requisitos previos de utilizar la API o CLI de {{site.data.keyword.cloud}} para crear nubes privadas virtuales (VPC). Para obtener más información, consulte [Guía de inicio](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) y [Configuración de VPC con API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Pasos de ejemplo
{: #vsrx-example-steps}

La topología para conectarse con el igual de Juniper vSRX remoto es similar a la [creación de una conexión de VPN entre dos VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc). Sin embargo, una de las partes de la conexión se sustituye por la unidad Juniper vSRX.

![especificar descripción de imagen aquí](./images/vpc-vpn-vsrx-figure.png)

### Para crear una conexión segura con un igual de Juniper vSRX remoto
{: #to-create-a-secure-connection-with-a-remote-vsrx-peer}

Cuando un igual de VPN recibe una solicitud de conexión de un igual de VPN remoto, utiliza los parámetros de fase 1 de IPsec para establecer una conexión segura y autenticar ese igual de VPN. A continuación, si la política de seguridad permite la conexión, la unidad Juniper vSRX establece el túnel utilizando los parámetros de fase 2 de IPsec y aplica la política de seguridad de IPsec. Los servicios de seguridad, autenticación y gestión de claves se negocian dinámicamente mediante el protocolo IKE.

**Para ofrecer soporte a estas funciones, la unidad Juniper vSRX debe realizar los pasos de configuración general siguientes:**

* Defina los parámetros de fase 1 que la unidad Juniper vSRX necesita para autenticar el igual remoto y establecer una conexión segura.
* Defina los parámetros de fase 2 que la unidad Juniper vSRX necesita para crear un túnel VPN con el igual remoto.
Para conectarse a la funcionalidad VPN de VPC de IBM Cloud se recomiendo la siguiente configuración:

1. Seleccione `IKEv1` en la fase 1;
2. Configure la modalidad de política en lugar de la modalidad de ruta;
3. Habilite `DH-group 2` en la propuesta de fase 1
4. Establezca `lifetime = 36000` en la propuesta de fase 1
5. Habilite PFS en la propuesta de fase 2
6. Establezca `lifetime = 10800` en la propuesta de fase 2
7. Introduzca la información de la subred y el igual en la propuesta de fase 2
8. Permita el tráfico UDP 500 en la interfaz externa.

#### Limitaciones conocidas
{: #vsrx-known-limitations}

* Juniper vSRX solo ofrece soporte a IKEv2 en la _modalidad de ruta_. Por lo tanto, si configura IKEv2 en la modalidad de política, verá el error `IKEv2 requiere una configuración de interfaz de enlace ya que solo se admite la configuración basada en ruta`. Sin embargo, la VPNaaS de la VPC de IBM Cloud actualmente solo ofrece soporte a la _modalidad de política_, por lo que debe configurar IKEv1 en la fase 1 para poder utilizar Juniper vSRX.

* La VPNaaS de la VPC de IBM Cloud inhabilita el PFS en la fase 2 de forma predeterminada y vSRX requiere que el PFS esté _habilitado_ en la fase 2. Por este motivo, debe crear una nueva política IPsec para sustituir la política predeterminada en el lado VPNaaS de VPC.

### Inicie sesión en Juniper vSRX para configurarlo utilizando SSH
{: #log-in-to-the-vsrx-to-configure-it-using-ssh}

#### A continuación, se muestra un ejemplo sobre cómo configurar la seguridad:
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

#### A continuación, se muestra un ejemplo sobre cómo configurar el cortafuegos:
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

Una vez que haya finalizado la ejecución del archivo de configuración, puede comprobar el estado de la conexión desde la CLI con el mandato siguiente:

```
 run show security ipsec security-associations
```
{: screen}

### Para crear una conexión segura con la VPC de IBM Cloud local
{: #vsrx-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Para crear una conexión segura, creará la conexión VPN dentro de la VPC, que es similar al ejemplo de 2 VPC.

Recuerde que debe habilitar PFS en la fase 2, por lo que debe crear una nueva política IPsec antes de crear una conexión.
{: important}

![vpc-vpn-vsrx-ipsec](./images/vpc-vpn-vsrx-ipsec.png)

Cree una pasarela de VPN en la subred de VPC junto con una conexión VPN entre la VPC y la unidad Juniper vSRX, estableciendo `local_cidrs` en el valor de subred de la VPC y `peer_cidrs` en el valor de subred de Juniper.

El estado de la pasarela aparece como `pendiente` mientras se crea la pasarela VPN y el estado pasa a ser `disponible` una vez completada la creación. La creación puede tardar un poco.
{:note}

![vpc-vpn-vsrx-connection](./images/vpc-vpn-vsrx-connection.png)

### Comprobar el estado de una conexión segura
{: #vsrx-check-the-status-for-a-secure-connection}

Puede comprobar el estado de la conexión a través de la consola de IBM Cloud. También puede intentar hacer `ping` de sitio a sitio utilizando los VSI.

![vpc-vpn-vsrx-status.png](./images/vpc-vpn-vsrx-status.png)
