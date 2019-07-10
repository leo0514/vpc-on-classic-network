---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, FortiGate, connection, secure

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creación de una conexión segura con un igual de FortiGate remoto
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

Este documento se basa en FortiGate 300C, versión de firmware v5.2.13, build762 (disponibilidad general).

Los pasos de ejemplo que siguen omiten los requisitos previos de utilizar la API o CLI de {{site.data.keyword.cloud}} para crear nubes privadas virtuales (VPC). Para obtener más información, consulte [Guía de inicio](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) y [Configuración de VPC con API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Pasos de ejemplo
{: #fortigate-example-steps}

La topología para conectarse con el igual de FortiGate remoto es similar a la creación de una conexión VPN entre dos VPC. Sin embargo, una de las partes se sustituye por la unidad 300C de FortiGate.

![especificar descripción de imagen aquí](./images/vpc-vpn-fg-figure.png)

### Para crear una conexión segura con un igual de FortiGate remoto
{: #to-create-a-secure-connection-with-the-remote-fortigate-peer}

Vaya a **VPN \> IPsec \> Túneles** y cree un nuevo túnel personalizado o edite uno existente.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

Cuando una unidad FortiGate recibe una solicitud de conexión de un igual de VPN remoto, utiliza los parámetros de fase 1 de IPsec para establecer una conexión segura y autenticar ese igual de VPN. A continuación, si la política de seguridad permite la conexión, la unidad FortiGate establece el túnel utilizando los parámetros de fase de IPsec y aplica la política de seguridad de IPsec. Los servicios de seguridad, autenticación y gestión de claves se negocian dinámicamente mediante el protocolo IKE.

**Para ofrecer soporte a estas funciones, la unidad de FortiGate debe realizar los pasos de configuración general siguientes:**

* Defina los parámetros de fase 1 que la unidad de FortiGate necesita para autenticar el igual remoto y establecer una conexión segura.

* Defina los parámetros de fase 2 que la unidad FortiGate necesita para crear un túnel VPN con el igual remoto.

* Cree políticas de seguridad para controlar los servicios permitidos y la dirección de tráfico permitida entre las direcciones IP de origen y de destino.

Se han establecido algunos parámetros de fase 1 y fase 2 típicos.

### Configuración para la VPN de VPC de IBM Cloud
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

Para conectarse a la funcionalidad VPN de VPC de IBM Cloud se recomiendo la siguiente configuración:

1. Elija IKEv2 en la autenticación;
2. Habilite `DH-group 2` en la propuesta de fase 1
3. Establezca `lifetime = 36000` en la propuesta de fase 1
4. Inhabilite el PFS en la propuesta de fase 2
5. Establezca `lifetime = 10800` en la propuesta de fase 2
6. Especifique la información de la subred en la fase 2
7. Los parámetros restantes conservan los valores predeterminados.

![especificar descripción de imagen aquí](./images/vpc-vpn-fg-network.JPG)

### Parámetros de red
{: #fortigate-network-parameters}

![especificar descripción de imagen aquí](./images/vpc-vpn-fg-authentication.JPG)

### Parámetros de autenticación
{: #fortigate-authentication-parameters}

![especificar descripción de imagen aquí](./images/vpc-vpn-fg-phase1.JPG)

### Parámetros de fase 1
{: #fortigate-phase-1-parameters}

![especificar descripción de imagen aquí](./images/vpc-vpn-fg-phase2.JPG)

### Parámetros de fase 2
{: #fortigate-phase-2-parameters}

## Para crear una conexión segura con la VPC de IBM Cloud local
{: #fortigate-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Para crear una conexión segura, creará la conexión VPN dentro de la VPC, que es similar al ejemplo de 2 VPC.

* Cree una pasarela de VPN en la subred de VPC junto con una conexión VPN entre la VPC y la unidad de FortiGate, estableciendo `local_cidrs` en el valor de subred de la VPC y `peer_cidrs` en el valor de subred de FortiGate.

**Nota:** El estado de la pasarela aparece como `pendiente` mientras se crea la pasarela VPN y el estado pasa a ser `disponible` una vez completada la creación. La creación puede tardar un poco.

![especificar descripción de imagen aquí](images/vpc-vpn-fg-connection.png)

### Comprobar el estado de la conexión segura
{: #fortigate-check-the-status-of-the-secure-connection}

Puede comprobar el estado de la conexión a través de la consola de IBM Cloud. También puede intentar hacer `ping` de sitio a sitio utilizando los VSI.

![especificar descripción de imagen aquí](images/vpc-vpn-fg-status.JPG)
