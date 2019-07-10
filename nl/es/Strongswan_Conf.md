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


# Creación de una conexión segura con un igual StrongSwan remoto
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

Este documento se basa en Strongswan, versión de Strongswan de Linux U5.3.5/K4.4.0-133-generic.

Los pasos de ejemplo que siguen omiten los requisitos previos de utilizar la API o CLI de {{site.data.keyword.cloud}} para crear nubes privadas virtuales (VPC). Para obtener más información, consulte [Guía de inicio](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) y [Configuración de VPC con API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Pasos de ejemplo
{: #strongswan-example-steps}

La topología para conectarse con el igual de StrongSwan remoto es similar a la creación de una conexión VPN entre dos VPC. Sin embargo, una de las partes de la conexión se sustituye por la unidad StrongSwan.

![especificar descripción de imagen aquí](./images/vpc-vpn-sw-figure.png)

### Para crear una conexión segura con un igual de StrongSwan remoto
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

Vaya a **/etc** y cree el nuevo archivo de configuración de túnel personalidad con un nombre similar a **ipsec.abc.conf**. Edite **/etc/ipsec.conf** para incluir **ipsec.abc.conf** y añada esta línea:

    include /etc/ipsec.abc.conf

Cuando un igual de VPN recibe una solicitud de conexión de un igual de VPN remoto, utiliza los parámetros de fase 1 de IPsec para establecer una conexión segura y autenticar ese igual de VPN. A continuación, si la política de seguridad permite la conexión, la unidad StrongSwan establece el túnel utilizando los parámetros de fase 2 de IPsec y aplica la política de seguridad de IPsec. Los servicios de seguridad, autenticación y gestión de claves se negocian dinámicamente mediante el protocolo IKE.

**Para ofrecer soporte a estas funciones, la unidad de StrongSwan debe realizar los pasos de configuración general siguientes:**

* Defina los parámetros de fase 1 que la unidad de StrongSwan necesita para autenticar el igual remoto y establecer una conexión segura.

* Defina los parámetros de fase 2 que la unidad de StrongSwan necesita para crear un túnel VPN con el igual remoto.
Para conectarse a la funcionalidad VPN de VPC de IBM Cloud se recomiendo la siguiente configuración:

1. Elija `IKEv2` en la autenticación;
2. Habilite `DH-group 2` en la propuesta de fase 1
3. Establezca `lifetime = 36000` en la propuesta de fase 1
4. Inhabilite el PFS en la propuesta de fase 2
5. Establezca `lifetime = 10800` en la propuesta de fase 2
6. Introduzca la información de la subred y el igual en la propuesta de fase 2

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

Establezca la clave precompartida en `/etc/ipsec.secrets`

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

Reinicie la unidad StrongSwan una vez que el archivo de configuración haya terminado de ejecutarse.

```
 ipsec restart
```
{: screen}

### Para crear una conexión segura con la VPC de IBM Cloud local
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* Para crear una conexión segura, creará la conexión VPN dentro de la VPC, que es similar al ejemplo de 2 VPC.

* Cree una pasarela de VPN en la subred de VPC, junto con una conexión VPN entre la VPC y la unidad StrongSwan, estableciendo `local_cidrs` en el valor de subred de la VPC y `peer_cidrs` en el valor de subred de StrongSwan.

El estado de la pasarela aparece como `pendiente` mientras se crea la pasarela VPN y el estado pasa a ser `disponible` una vez completada la creación. La creación puede tardar un poco.
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### Comprobar el estado de una conexión segura
{: #strongswan-check-the-status-for-a-secure-connection}

Puede comprobar el estado de la conexión a través de la consola de IBM Cloud. También puede intentar hacer `ping` de sitio a sitio utilizando los VSI.

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
