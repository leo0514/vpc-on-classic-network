---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: VPC, subnet, address prefixes, design, addressing

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


# Diseño de un plan de direcciones para una VPC 
{: #vpc-addressing-plan-design}

El primer paso en el diseño de una VPC debe ser el diseño del plan de direcciones. Un plan de direcciones ejecutado correctamente tiene dos objetivos:

* Se ajusta a los requisitos de comunicación de las instancias de VPC.
* Conserva la flexibilidad para el futuro crecimiento. 

Este documento proporciona un ejemplo de diseño del plan de direcciones para una aplicación web de tres niveles, en la que cada nivel está soportado por varias zonas.

Aunque cada VPC de {{site.data.keyword.cloud}} se despliega en una región específica, la VPC puede abarcar todas las zonas de esa región. La VPC de {{site.data.keyword.cloud_notm}} define un prefijo de dirección predeterminado para cada zona. Estos prefijos de dirección permiten la comunicación entre las instancias de {{site.data.keyword.cloud_notm}} en zonas distintas, proporcionando la información de direccionamiento relacionada con la zona que necesita el [direccionador implícito](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#implicit-router).

Se utilizan los mismos pasos de diseño, independientemente de si la aplicación está contenida completamente en la nube, o si partes de la aplicación se ejecutan en otra ubicación.
{: tip}

## Consideraciones y supuestos de diseño
{: #design-considerations-and-assumptions}

Al diseñar el plan de direccionamiento para una aplicación, la consideración principal es mantener los bloques CIDR utilizados para crear subredes dentro de una única zona lo más contiguos posible. Así, el diseñador permite resumirlos en un solo prefijo de dirección, permitiendo el futuro crecimiento.

Otra consideración es el número de direcciones disponibles que una subred puede necesitar para el escalado horizontal. En la tabla siguiente se muestra el número de direcciones disponibles en una subred, basado en el tamaño de bloque CIDR especificado:

| Tamaño de bloque CIDR | Direcciones disponibles |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Con base en estas dos consideraciones, éstas son las suposiciones que hacemos para este ejemplo:

* Para este ejemplo, se utilizarán rangos CIDR del bloque 172.16.0.0/12 de direcciones de RFC 1918 para todas las subredes.
* Damos por supuesto que la capa de presentación de la aplicación es un revestimiento ligero sobre una API REST. Por lo tanto, el escalado horizontal afecta a la capa media más que a la capa de presentación.

## Determine el tamaño de subred de cada nivel
{: #determine-each-tier-s-subnet-size}

El paso siguiente consiste en determinar el tamaño de subred de cada nivel (en términos de direcciones disponibles). Cada nivel de la aplicación tiene presencia en cada zona, por lo tanto, cada zona necesitará tres subredes.

A continuación se muestran las consideraciones que utilizamos para planificar el tamaño de subred de cada nivel:

* El nivel de la base de datos (el programa de fondo) es el que con menos probabilidad necesitará escalado dinámico, por lo que estas subredes pueden ser las más pequeñas. Es decir, estas subredes pueden contener el menor número de direcciones disponibles. 
    * _En este se ejemplo utiliza un bloque CIDR `/27`, que permite 27 direcciones en este nivel._
* El nivel medio es el que más probablemente necesita el escalado dinámico, por lo que estas subredes serán las más grandes. Es decir, deben contener el mayor número de direcciones disponibles. 
    * _En este se ejemplo utiliza un bloque CIDR `/25`, que permite 123 direcciones en este nivel._
* El nivel frontal queda entremedio; no necesita tantas direcciones como el nivel medio, pero necesita más que el nivel de base de datos. 
    * _Este ejemplo utiliza un bloque CIDR `/26`, que permite ver 59 direcciones en este nivel._

## Combinación de las subredes y selección de los prefijos de dirección
{: #combining-the-subnets-and-selecting-the-address-prefixes}

Para seleccionar un prefijo de dirección aceptable para cada zona, necesitará un tamaño de subred que sea lo suficientemente grande como para acomodar las tres subredes de cada nivel, y aún así dejar espacio para el escalado horizontal y una futura expansión. 

Un prefijo de dirección `/24` es el prefijo más pequeño en el que se pueden combinar estas tres subredes (27 + 123 + 59). Se recomienda seleccionar el siguiente tamaño de subred más grande, no el más pequeño. Asignar el siguiente tamaño de subred más grande (`/23`) permite el escalado horizontal más allá de los límites dados anteriormente, porque permite añadir nuevas subredes a cada capa, desde dentro del mismo prefijo de dirección.

Ahora que hemos determinado el tamaño de subred correcto, se pueden asignar los prefijos de direcciones, uno para cada zona:

|  Zona  | Prefijo de dirección  |
| ------ | --------------- |
| Zona 1 | `172.16.0.0/23` |
| Zona 2 | `172.16.2.0/23` |
| Zona 3 | `172.16.4.0/23` |

Y, a partir de esta base, se pueden asignar las 3 subredes dentro de cada zona:

|  Zona  |   Nivel   |    CIDR de subred    |
| ------ | -------- | ----------------- |
| Zona 1 |  Medio  |  `172.16.0.0/25`  |
| Zona 1 |  Frontal   |  `172.16.1.0/26`  |
| Zona 1 | Base de datos | `172.16.1.128/27` |
| Zona 2 |  Medio  |  `172.16.2.0/25`  |
| Zona 2 |  Frontal   |  `172.16.3.0/26`  |
| Zona 2 | Base de datos | `172.16.3.128/27` |
| Zona 3 |  Medio  |  `172.16.4.0/25`  |
| Zona 3 |  Frontal   |  `172.16.5.0/26`  |
| Zona 3 | Base de datos | `172.16.5.128/27` |

## Consideraciones para ampliar una infraestructura existente
{: #considerations-for-extending-an-existing-infrastructure}

Al planificar una VPC que amplíe una infraestructura existente, puede seguir los pasos anteriores, tanto si la infraestructura es su infraestructura local, o si es otra VPC o incluso otra nube. Recuerde que no debe reutilizar los rangos de direcciones existentes. Evitando la reutilización de direcciones, puede maximizar su capacidad para beneficiarse de las características de {{site.data.keyword.cloud_notm}} VPC.
