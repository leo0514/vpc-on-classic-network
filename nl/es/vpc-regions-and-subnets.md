---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-07"

keywords: vpc, address prefix, region, subnet, zone, reserved, IP, ranges, deleting, creating, CIDR

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# Rangos de direcciones IP, prefijos de direcciones, regiones y subredes
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (tema de ayuda enlazado)

En este documento se tratan las relaciones entre regiones, prefijos de direcciones y subredes para {{site.data.keyword.cloud}} VPC:

* Las VPC se despliegan y se enlazan a una región.
* Dentro de esa región, las VPC pueden abarcar varias zonas.
* Los prefijos de dirección permiten la comunicación entre las zonas que una VPC abarca. Cada VPC obtiene un prefijo de dirección predeterminada de acceso directo para cada zona que abarca.
* Las subredes se crean dentro del ámbito de un prefijo de dirección, lo que significa que una subred debe estar contenida completamente dentro de un solo prefijo de dirección existente.
* Si utiliza un rango de IP fuera de los rangos definidos en [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` o en `192.168.0.0/16`) para una subred, es posible que las instancias conectadas a esa subred no puedan acceder a partes de la Internet pública.

## Regiones y VPC de IBM Cloud
{: #ibm-cloud-vpc-and-regions}

Una VPC de {{site.data.keyword.cloud}} se despliega en una región, pero puede abarcar varias zonas. Cada región contiene varias zonas, que representan dominios de error independientes.

## Prefijos de dirección y VPC de IBM Cloud
{: #ibm-cloud-vpc-and-address-prefixes}

Los prefijos de dirección permiten la comunicación entre las instancias de {{site.data.keyword.cloud_notm}} VPC en zonas distintas. Proporcionan información de direccionamiento, que necesita el _direccionador implícito_, para enviar datos a la zona donde se encuentra la instancia de destino. Cada subred debe estar contenida dentro de un prefijo de dirección. {{site.data.keyword.cloud_notm}} VPC no admite el soportar de subredes "locales" o "inaccesibles".

El _direccionador implícito_ es la conectividad de red inherente entre todas las subredes creadas dentro de una VPC.
{: note}

Cada VPC de {{site.data.keyword.cloud_notm}} puede tener hasta cinco prefijos de dirección para cada zona. Para ayudarle a empezar, {{site.data.keyword.cloud_notm}} VPC define un prefijo de dirección predeterminado para cada zona (consulte la tabla a continuación), no obstante, recomendamos diseñar un plan de direcciones de VPC antes de desplegar una.

### Prefijos de dirección predeterminados de VPC
{: #default-vpc-address-prefixes}

Cuando se crea una nueva VPC, se asignan prefijos de dirección predeterminados como se indica a continuación, en función de la región y la zona.

[las VPC de acceso clásico](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) tienen un conjunto de prefijos de dirección predeterminados distinto.
{: important}

Zona         | Prefijo de dirección
---------------|---------------
`us-south-1`   | `10.240.0.0/18`
`us-south-2`   | `10.240.64.0/18`
`us-south-3`   | `10.240.128.0/18`
`eu-de-1`      | `10.243.0.0/18`
`eu-de-2`      | `10.243.64.0/18`
`eu-de-3`      | `10.243.128.0/18`
`jp-tok-1`     | `10.244.0.0/18`
`jp-tok-2`     | `10.244.64.0/18`
`jp-tok-3`     | `10.244.128.0/18`

A las nuevas zonas o regiones se les asignarán distintos prefijos predeterminados.

### Prefijos de dirección y la interfaz de usuario de la consola de IBM Cloud
{: #address-prefixes-and-the-ibm-cloud-console-ui}

Cuando crea una VPC utilizando la interfaz de usuario de la consola de IBM Cloud, el sistema selecciona el prefijo de dirección de manera automática y le solicita que cree una subred en el prefijo predeterminado. Si el esquema de dirección no se adapta a sus requisitos, puede personalizar los prefijos de dirección después de crear la VPC. A continuación, puede crear subredes en los prefijos de direcciones personalizados y suprimir la subred que ha creado con el prefijo predeterminado.

Este método alternativo es necesario para utilizar BYOIP a través de la interfaz de usuario de la consola de IBM Cloud.
{:note}

## Subredes y VPC de IBM Cloud
{: #ibm-cloud-vpc-and-subnets}

Puede dividir una VPC de {{site.data.keyword.cloud_notm}} en subredes. Todas las subredes de una VPC de {{site.data.keyword.cloud_notm}} pueden estar en contacto entre ellas mediante el direccionamiento L3 privado realizado por un direccionador implícito. No es necesario configurar ningún direccionador ni ninguna ruta.

Datos útiles sobre las subredes en VPC:

* Una subred consta de un rango de direcciones IP que debe especificar.
* Una subred está enlazada a una sola zona y no puede abarcar varias zonas o regiones.
* Una subred puede abarcar la totalidad de la zona dentro de una VPC de {{site.data.keyword.cloud_notm}}.
* Debe crear la VPC antes de crear las subredes en dicha VPC.
* El soporte de IPv6 no está disponible.
* Puede asociar o desasociar una VSI a una subred. (Requiere que añada una vNIC y seleccione un ancho de banda.)
* Cada subred debe estar contenida dentro de un prefijo de dirección que pertenezca a la zona en la que está enlazada dicha subred.

Puede utilizar su propio rango de dirección IPv4 público (BYOIP) en su cuenta de {{site.data.keyword.cloud_notm}} VPC. Cuando utilice BYOIP, {{site.data.keyword.cloud_notm}} deberá configurar las direcciones IPv4 en los recursos de {{site.data.keyword.cloud_notm}}, que enviarán paquetes a y desde las direcciones proporcionadas. Por lo tanto, como resultado de utilizar el rango IPv4 proporcionado en {{site.data.keyword.cloud_notm}}, las direcciones IP pueden estar expuestas al personal de soporte de IBM y a terceros como parte de su uso de este servicio.
{:important}

![Visión general de IBM Cloud VPC](images/vpc-experience.svg "Visión general de IBM Cloud VPC")

### Utilización de prefijos de dirección para subredes
{: #using-address-prefixes-for-subnets}

Cada subred debe existir dentro de un prefijo de dirección.
 * Para la nueva subred, puede elegir el rango de direcciones IP a partir de los prefijos de dirección existentes.
 * Si los prefijos de dirección de la zona no son adecuados, es posible editar uno de los prefijos de dirección existentes, o añadir uno nuevo a la zona (utilizando la API, CLI o IU).

### Direcciones IP disponibles
{: #available-ip-addresses}

Estas son las direcciones IP disponibles, tal como se definen en **RFC 1918**:

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

Si utiliza un rango de IP que queda fuera de los rangos permitidos para una subred (indicados en secciones anteriores), es posible que las instancias conectadas a esa subred no puedan acceder a partes de la Internet pública.

### Más información sobre la creación de una subred
{: #more-about-creating-a-subnet}

Puede especificar una subred para la VPC de dos maneras:
  * Puede crear una subred proporcionando el tamaño de la subred que necesita, como el número de direcciones soportadas (por ejemplo 1024)
  * Puede crear una subred proporcionando un rango de CIDR (como por ejemplo 10.0.0.8/29)

A modo de ejemplo de cómo especificar un bloque 1024 utilizando CIDR, si especifica un rango de CIDR en lugar de un tamaño de subred, el bloque IPv4 `192.168.100.0/22` representa las direcciones IPv4 1024 de `192.168.100.0` a `192.168.103.255`.
{:tip}

