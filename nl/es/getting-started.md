---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-06-06"

keywords: provisioning, resources, permissions

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Guía de aprendizaje de iniciación
{: #getting-started}

Para empezar a trabajar con las redes para {{site.data.keyword.cloud}} Virtual Private Cloud:

1. Cree una nube privada virtual.
2. Cree una o varias subredes en la nube privada virtual en una o varias zonas.
3. Cree una pasarela pública (PGW) en una subred si desea que los recursos de la subred tengan acceso a Internet o viceversa.

## Requisitos previos

 * **Permisos de usuario**: Asegúrese de que el usuario tiene permisos suficientes para crear y gestionar recursos en la VPC. Para obtener una lista de los permisos necesarios, consulte [Cómo otorgar los permisos necesarios para usuarios de VPC](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources).

## Utilice la interfaz de usuario, la CLI o la API REST para suministrar recursos de red de VPC

El suministro y la gestión de los recursos de red de VPC se pueden realizar a través de la interfaz de usuario, la CLI o la API REST.

* Para acceder a través de la interfaz de usuario, inicie una sesión en la [consola de IBM Cloud ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")]( https://{DomainName}/vpc){: new_window}. Siga la [guía de la interfaz de usuario](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console) si necesita ayuda.
* Para utilizar la interfaz de línea de mandatos, siga el ejemplo [Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).
* Los usuarios más avanzados pueden llamar directamente a las [API REST](https://{DomainName}/apidocs/vpc-on-classic). Siga la guía de aprendizaje de [código de ejemplo](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) para comenzar a utilizar las API REST.

## Siguientes pasos

Después de suministrar la red VPC (Virtual Private Cloud), siga explorando.

* [Creación y gestión de instancias de servidor virtual](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [Utilización de grupos de seguridad](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [Utilización de ACL de red](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [Utilización de VPN](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [Utilización de equilibradores de carga](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
