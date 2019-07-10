---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: VRF, router, hypervisor, address prefixes, classic access, implicit router, packet flows, NAT, data flows

subcollection: vpc-on-classic-network


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}

# VPC detrás de la cortina
{: #vpc-behind-the-curtain}

Esta página presenta una imagen conceptual más detallada de lo que está sucediendo "detrás de la cortina" con respecto a la red de VPC. Se espera que los lectores tengan ciertos conocimientos sobre redes.

## Aislamiento de red
{: #network-isolation}

El aislamiento de red de VPC tiene lugar en tres niveles:

* **Hipervisor**: las VSI (instancias de servidor virtual) están aisladas por el propio hipervisor. Una VSI no puede acceder directamente a otras VSI alojadas por el mismo hipervisor si no están en la misma VPC.

* **Red**: el aislamiento se produce a nivel de red mediante el uso de **identificadores de red virtual** (VNI). Estos identificadores se circunscriben a la zona local. Estos VNI se añaden a todos los paquetes de datos que entran en cualquier zona de la VPC: entrando, bien desde el hipervisor, cuando los envía un VSI, o entrando en la zona desde la nube, cuando los envía la función de direccionamiento implícito.

A cada paquete que sale de la zona se le quita el VNI. Cuando el paquete llega a su zona de destino, entrando a través de la función de direccionamiento implícito, el direccionador implícito siempre añade el VNI adecuado para esa zona.
{: note}

* **Direccionador**: la _función de direccionador implícito_ ofrece aislamiento a cada VPC mediante el suministro de una **función de direccionamiento virtual** (VRF) y una VPN con MPLS (conmutación de etiqueta multiprotocolo) en la red troncal de la nube. La VRF de cada VPN tiene un identificador exclusivo y su aislamiento permite que cada VPC tenga acceso a su propia copia del espacio de direcciones IPv4. La VPN de MPLS permite federar todos los extremos de la nube: infraestructura clásica, Direct Link y VPC.

## Prefijos de dirección
{: #address-prefixes}

Los prefijos de dirección son la información de resumen que utiliza la función de direccionamiento implícito de VPC para localizar una _VSI de destino_, independientemente de la zona de disponibilidad en la que se encuentra la VSI de destino. La función principal de los prefijos de dirección es optimizar el direccionamiento a través de la VPN de MPLS, evitando al mismo tiempo los casos de direccionamiento patológico. Todas las subredes creadas en una VPC deben estar contenidas en un prefijo de dirección, de modo que todos los VSI de una VPC resulten accesibles desde todas las otras VSI de la VPC.

## Flujos de paquetes de datos y el direccionador implícito
{: #data-packet-flows-and-the-implicit-router}

Se producen seis tipos diferentes de flujos de paquetes de datos de VSI en una VPC. En orden ascendente de complejidad, estos flujos son:

* Dentro de la subred, dentro del host (mismo hipervisor)
* Dentro de la subred, entre hosts
* Entre subredes, dentro de la zona
* Entre subredes, entre zonas
* Servicio extra-VPC (para el acceso a IaaS o CSE)
* Internet extra-VPC (para el acceso a Internet)

Flujos de datos **dentro de la subred, dentro del host**: son los más sencillos. Los paquetes fluyen entre las VSI en el hipervisor y no ningún paquete abandona el hipervisor.

Flujos de datos **dentro de la subred, entre hosts**: estos flujos incluyen paquetes que abandonan el hipervisor. Cada paquete se etiqueta con el VNI (identificador de red virtualizado) adecuado para garantizar el aislamiento de los datos, y luego se envía al hipervisor de destino que aloja la VSI de destino. El hipervisor de destino abre el VNI y reenvía el paquete de datos a la VSI de destino.

Flujos de datos **entre subredes y dentro de la zona**: estos flujos incluyen los paquetes que utilizan la función de direccionador implícita de la VPC, que conecta todas las subredes creadas en la VPC. Direcciona el paquete de datos al hipervisor de destino correcto. Si el hipervisor de destino es distinto del hipervisor de origen, el paquete de datos se etiqueta con el VNI adecuado y se envía al hipervisor de destino. Allí, se abre el VNI y el paquete de datos se reenvía a la VSI de destino. (Estos últimos pasos coinciden con los del paso anterior de flujo de datos.)

Flujos de datos **entre subredes, entre zonas**: para estos flujos, la función del direccionador implícito elimina el VNI y reenvía el paquete a al VPN de MPLS de la VPC para que lo dirija por la red troncal de la nube. En la zona de destino, la función de direccionador implícito etiqueta el paquete de datos con el VNI adecuado. Luego el paquete se reenvía al hipervisor de destino, donde el VNI se vuelve a quitar (tal como se ha descrito anteriormente) para que el paquete de datos se pueda reenviar a la VSI de destino.

Flujos de datos del **Servicio extra-vpc**: los paquetes destinados a los servicios IaaS o IBM Cloud Service Endpoint (CSE) utilizan la función del direccionador implícito de la VPC y también utilizan una función de conversión de direcciones de red (NAT). La función de conversión sustituye la dirección de la VSI por la dirección IPv4, que identifica la VPC ante el servicio IaaS o CSE que se ha solicitado.

Flujos de datos de **Internet extra-vpc**: los paquetes destinados a Internet son los más complejos. Además de utilizar la función de direccionador implícito de la VPC, cada uno de estos flujos también utiliza una de las dos funciones de conversión de direcciones de red (NAT) del direccionador implícito:

  * una NAT explícita entre uno y muchos a través de una función de pasarela pública que da servicio a todas las subredes conectadas a la misma.
  * NAT de uno a uno asignado a VSI individuales.

Después de la conversión de NAT, el direccionador implícito reenvía estos paquetes destinados a Internet a Internet, utilizando la red troncal de la nube.

## Acceso clásico
{: #classic-access}

La característica de [**Acceso clásico**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc) para VPC se consigue reutilizando el identificador de VRF de la cuenta de la infraestructura clásica de {{site.data.keyword.cloud}} como identificador de VRF para la VPC. Esta implementación permite que la función del direccionador implícito de VPC se una a la misma VPN de MPLS que utiliza la cuenta de infraestructura clásica. Por lo tanto, la VPC tiene acceso a los recursos clásicos y a todo a lo que se puede acceder mediante las conexiones existentes de Direct Link.
