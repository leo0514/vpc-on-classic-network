---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-29"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Seguridad en la VPC de IBM Cloud
{: #security-in-your-ibm-cloud-vpc}

Puede mantener la VPC y las cargas de trabajo seguras controlando el tráfico de red utilizando grupos de seguridad (SG), listas de control de acceso de red (ACL) o ambos tipos de control. Recuerde que:

* Los grupos de seguridad controlan el tráfico en cada instancia (VSI).
* Las listas de control de acceso controlan el tráfico en cada subred.

## Visión general de la seguridad
{: #security-overview}

* El tráfico hacia y desde una subred se puede controlar mediante las listas de control de acceso (ACL)
* Los grupos de seguridad (SG) pueden controlar el tráfico en el nivel de VSI
* Configure una pasarela pública para el acceso de la subred a Internet, protegida por las ACL
* Configure una IP flotante para el acceso de VSI a Internet, protegida por los SG

![Conectividad y seguridad de IBM VPC](images/vpc-connectivity-and-security.svg "Conectividad y seguridad de IBM VPC")

## Definiciones
{: #definitions}

Este [glosario](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) proporciona definiciones y descripciones de las ACL y los SG y las acciones que pueden realizar los mismos. La sección siguiente describe la funcionalidad básica de las ACL y los grupos de seguridad y explica las formas en que la VPC ofrece soporte para el cifrado de extremo a extremo.

### Lista de control de acceso
{: #access-control-list}

Una **Lista de control de acceso (ACL)** puede gestionar (es decir, puede permitir o denegar) el tráfico de entrada y salida de una subred. Una ACL no tiene estado, lo que significa que las reglas de entrada y salida deben especificarse de forma independiente y explícita. Cada ACL consta de reglas, basadas en una *IP de origen*, un *puerto de origen*, una *IP de destino*, un *puerto de destino* y un *protocolo*.

En la VPC de {{site.data.keyword.cloud}}, cada subred se crea con una ACL predeterminada, lo que permite el tráfico de entrada y salida, aunque los clientes pueden crear ACL personalizadas. Solo una ACL está conectada a un subred en cualquier momento, aunque es posible conectar una ACL a varias subredes. Para obtener más información sobre cómo utilizar las ACL, consulte nuestra [guía de ACL](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls).

### Grupo de seguridad
{: #security-group}

Un **grupo de seguridad** actúa como un cortafuegos virtual que controla el tráfico de uno o varios servidores (VSI). Un grupo de seguridad es una recopilación de reglas que especifican si se debe permitir el tráfico en una VSI asociada. 

Cuando un cliente crea una VSI, puede asociar uno o varios grupos de seguridad a dicha VSI. Si se proporcionan los permisos adecuados, los clientes pueden modificar las reglas del grupo de seguridad utilizando la consola de IBM, la CLI o la API.

Para obtener más información sobre cómo crear una VSI que utilice grupos de seguridad y sobre otras prestaciones de los grupos de seguridad, consulte nuestra [guía sobre los grupos de seguridad](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups).

### Cifrado de principio a fin
{: #end-to-end-encryption}

Aunque la VPC de IBM Cloud no proporciona un cifrado de principio a fin, lo permite. Por ejemplo, si tiene un punto final seguro en un servidor virtual (por ejemplo, un servidor HTTPS en el puerto 443), puede adjuntar una IP flotante a dicho servidor y la conexión se cifrará de principio a fin desde el cliente al servidor del puerto 443.  Nada de lo que hay en la vía de acceso fuerza un descifrado.

Sin embargo, debe tener en cuenta que si utiliza un protocolo inseguro como HTTP en el puerto 80, los datos estarán en claro de principio a fin.
