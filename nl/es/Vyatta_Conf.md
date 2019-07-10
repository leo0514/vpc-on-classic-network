---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Vyatta, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creación de una conexión segura con un igual Vyatta remoto
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

Este documento se basa en la versión de Vyatta: AT&T vRouter 5600 1801d.

Los pasos de ejemplo que siguen omiten los requisitos previos de utilizar la API o CLI de {{site.data.keyword.cloud}} para crear nubes privadas virtuales (VPC). Para obtener más información, consulte [Guía de inicio](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) y [Configuración de VPC con API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Pasos de ejemplo
{: #vyatta-example-steps}

La topología para conectarse con el igual de Vyatta remoto es similar a la creación de una conexión de VPN entre dos VPC de {{site.data.keyword.cloud_notm}}. Sin embargo, una de las partes se sustituye por la unidad Vyatta.

![especificar descripción de imagen aquí](images/vpc-vpn-vy-figure.png)

### Para crear una conexión segura con un igual de Vyatta remoto
{: #vyatta-to-create-a-secure-connection-with-the-remote-vyatta-peer}

Cuando un igual de VPN recibe una solicitud de conexión de un igual de VPN remoto, utiliza los parámetros de fase 1 de IPsec para establecer una conexión segura y autenticar ese igual de VPN. A continuación, si la política de seguridad permite la conexión, la unidad Vyatta establece el túnel utilizando los parámetros de fase 2 de IPsec y aplica la política de seguridad de IPsec. Los servicios de seguridad, autenticación y gestión de claves se negocian dinámicamente mediante el protocolo IKE.

Puede dirigirse a la línea de mandatos de Vyatta para configurar el túnel IPsec o puede cargar un archivo `*.vcli` para cargar la configuración.

**Para ofrecer soporte a estas funciones, la unidad Vyatta debe realizar los pasos de configuración general siguientes:**

* Defina los parámetros de fase 1 que la unidad Vyatta necesita para autenticar el igual remoto y establecer una conexión segura.

* Defina los parámetros de fase 2 que la unidad Vyatta necesita para crear un túnel VPN con el igual remoto.

Para conectarse a la funcionalidad VPN de VPC de IBM Cloud se recomiendo la siguiente configuración:

1. Elija `IKEv2` en la autenticación;
2. Habilite `DH-group 2` en la propuesta de fase 1
3. Establezca `lifetime = 36000` en la propuesta de fase 1
4. Inhabilite el PFS en la propuesta de fase 2
5. Establezca `lifetime = 10800` en la propuesta de fase 2
6. Introduzca la información de iguales y subredes en la fase 2

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.161.151_test_ike
set security vpn ipsec ike-group 169.61.161.151_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.161.151_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.161.151_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.161.151_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.161.151_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.161.151_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.161.151 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.161.151 authentication pre-shared-secret ******
set security vpn ipsec site-to-site peer 169.61.161.151 ike-group 169.61.161.151_test_ike
set security vpn ipsec site-to-site peer 169.61.161.151 default-esp-group 169.61.161.151_test_ipsec
set security vpn ipsec site-to-site peer 169.61.161.151 description "automation test"
set security vpn ipsec site-to-site peer 169.61.161.151 local-address 169.45.74.119
set security vpn ipsec site-to-site peer 169.61.161.151 connection-type initiate


set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 local prefix 192.168.200.0/24
set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 remote prefix 192.168.17.0/28

commit
```
{: screen}

A continuación, puede cargar el archivo `*.vcli` en Vyatta utilizando el SCP para aplicar la configuración.

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### Para crear una conexión segura con la VPC de IBM Cloud local
{: #vyatta-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

 Para crear una conexión segura, creará la conexión VPN dentro de la VPC, que es similar al ejemplo de 2 VPC.

* Cree una pasarela de VPN en la subred de VPC junto con una conexión VPN entre la VPC y la unidad Vyatta, estableciendo `local_cidrs` en el valor de subred de la VPC y `peer_cidrs` en el valor de subred de Vyatta.

El estado de la pasarela aparece como `pendiente` mientras se crea la pasarela VPN y el estado pasa a ser `disponible` una vez completada la creación. La creación puede tardar un poco.
{: note}

![especificar descripción de imagen aquí](images/vpc-vpn-vy-connection.png)

### Comprobar el estado de la conexión segura
{: #vyatta-check-the-status-of-the-secure-connection}

Puede comprobar el estado de la conexión a través de la consola de {{site.data.keyword.cloud_notm}}. También puede intentar hacer `ping` de sitio a sitio utilizando los VSI.

![especificar descripción de imagen aquí](images/vpc-vpn-vy-status.png)
