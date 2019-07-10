---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: capabilities, use cases, subnets, VPN, connections, reserved, IP, IPv4, floating

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Acerca de las redes para VPC
{: #about-networking-for-vpc}

En este documento encontrará conceptos que describen casos de uso y prestaciones en la VPC de {{site.data.keyword.cloud}} para subredes, direcciones IP flotantes y conexiones VPN.

![Conectividad y seguridad de IBM VPC](images/vpc-connectivity-and-security.svg "Conectividad y seguridad de IBM VPC")

Tal como se muestra en la figura:

* Las subredes se pueden conectar a Internet público a través de una pasarela pública opcional (PGW).
* Puede asignar una dirección IP flotante (FIP) a cualquier VSI para llegar a ella desde Internet o viceversa.
* Las subredes de la VPC de IBM Cloud ofrecen conectividad privada; pueden comunicarse entre sí a través de un enlace privado. No es necesario configurar ninguna ruta.
* Para obtener más información, consulte el tema [Acerca de la infraestructura de VPC](/docs/vpc-on-classic?topic=vpc-on-classic-about).

## Terminología
{: #terminology}

Este [glosario](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) contiene definiciones e información sobre los términos utilizados en este documento para la VPC de IBM Cloud.

## Características de las subredes en la VPC
{: #characteristics-of-subnets-in-the vpc}

Una subred consta de un rango de direcciones IP especificado (bloque CIDR). Las subredes están enlazadas a una única zona y no pueden abarcar varias zonas o regiones. Sin embargo, una subred puede abarcar la totalidad de las abstracciones de la zona dentro de su nube privada virtual. Las subredes en la misma VPC de IBM Cloud están conectadas entre sí.

### Zonas
{: #zones}

Una zona es una abstracción diseñada para ayudar a mejorar la tolerancia al error y la latencia disminuida. Una zona garantiza las propiedades siguientes:

 * Cada zona es un dominio independiente de tolerancia a errores y es extremadamente improbable que dos zonas de una región fallen simultáneamente
 * El tráfico entre las zonas de una región tendrá < 2 ms de latencia

### Direcciones IP reservadas
{: #reserved-ip-addresses}

Algunas direcciones IP están reservadas para que IBM las utilice al trabajar con la nube privada virtual. A continuación, se muestran las direcciones IP reservadas (las direcciones IP especificadas presuponen que el rango de CIDR de la subred es 10.10.10.0/24):

  * Primera dirección en el rango de CIDR (10.10.10.0): Dirección de red
  * Segunda dirección en el rango de CIDR (10.10.10.1): Dirección de pasarela
  * Tercera dirección en el rango de CIDR (10.10.10.2): reservado por IBM
  * Cuarta dirección en el rango de CIDR (10.10.10.3): reservado por IBM para su uso futuro
  * Última dirección en el rango CIDR (10.10.10.255): Dirección de difusión de red

### Utilizar una pasarela pública para la conectividad externa de una subred
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

Una **Pasarela pública (PGW)** permite a una subred (con todos los VSI adjuntos a la subred) conectarse a Internet. Tenga en cuenta que las subredes son privadas de forma predeterminada; sin embargo, puede crear opcionalmente una pasarela pública y conectar una subred a la misma. Una vez adjuntada la subred a la pasarela pública, todos los VSI de dicha subred se podrán conectar a Internet.

La pasarela pública utiliza _una conversión de direcciones de red de muchas a una_, lo que significa que miles de VSI con direcciones privadas utilizarán 1 dirección IP pública para comunicarse con Internet público. La pasarela pública no permite que Internet inicie una conexión con estas instancias. Utilice la API para adjuntar y desconectar subredes a y desde la pasarela pública.

La figura siguiente resume el ámbito de los servicios de pasarela.

![servicios de pasarela](images/scope-of-gateway-services.png)

### Utilice una dirección IP flotante para la conectividad externa de una VSI
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

Las **direcciones IP flotantes** son direcciones IP que proporciona el sistema y son accesibles desde la Internet pública.

Puede reservar una dirección IP flotante desde la agrupación de direcciones IP flotantes disponibles proporcionada por IBM y puede asociarla o desasociarla con cualquier instancia en la misma nube privada virtual, mediante la vNIC de dicha instancia. Es posible asociar cualquier dirección IP flotante a varios tipos de instancias de servidor virtual (VSI), por ejemplo, un equilibrador de carga o una pasarela VPN.

La dirección IP flotante no se puede asociar con varias interfaces. Debe especificar la interfaz en la VSI que se asociará con dicha IP flotante individual. Esta interfaz también tendrá una dirección IP privada. El sistema de fondo realizar operaciones de _conversión de direcciones de red de una a una_ entre la IP flotante y la IP privada de la interfaz. Solo se libera una IP flotante cuando solicita específicamente la acción de liberación. 

**Notas:**
* **La asociación de una dirección IP flotante con una VSI elimina la VSI de la conversión de direcciones de red de muchas a una de la pasarela pública mencionada anteriormente.**
* **Actualmente, la IP flotante solo da soporte a direcciones IPv4.**
* **No puede traer su propia dirección IP pública para utilizarla como IP flotante.**

Para obtener más información sobre las operaciones de NAT, consulte [el documento RFC de Internet relacionado ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Utilice una VPN para una conectividad externa segura
{: #use-a-vpn-for-secure-external-connectivity}

El servicio de red privada virtual (VPN) está disponible para que los usuarios se conecten a su VPC de IBM Cloud desde Internet de forma segura. Para obtener instrucciones paso a paso, consulte nuestra [Guía de IU de IBM Console](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console).

**Prestaciones de VPN**
  * Capacidad de crear, recuperar, actualizar y suprimir un servicio VPN (VPN de IPSEC de sitio a sitio) para gestionar la transferencia de datos.
  * Capacidad para asociar un servicio VPN a la red virtual o la VPC de IBM Cloud de un cliente.
  * Capacidad de identificar las subredes locales del cliente ya sea mediante definición estática o mediante direccionamiento dinámico.
  * Soporte para cifrados seguros como, por ejemplo, SHA256, AES, 3DES, IKEv2
  * Fiabilidad de servicio incorporada.
  * Supervisión del estado de la conexión VPN.
  * Soporte para los modos de iniciador y respondedor; es decir, el tráfico se puede iniciar desde cualquier lado del túnel.
