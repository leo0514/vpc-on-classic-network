---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: peering, classic, infrastructure, VRF, resources

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

# Peering clásico para VPC
{: #setting-up-classic-peering-for-vpc}

{{site.data.keyword.cloud}} VPC puede establecer una conexión de igual a igual (peering) con los recursos clásicos de IBM Cloud. Las cuentas deben cumplir con algunos requisitos para poder configurar peering.

Para ver instrucciones paso a paso para crear una VPC con peering clásico habilitado, consulte nuestro [documento sobre la infraestructura de VPC](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc), donde se muestra este procedimiento en detalle.

## Requisitos
{: #classic-peering-requirements}

1. La VPC se debe haber designado para "peering clásico" en el momento de la creación, no se puede designar posteriormente.

2. La cuenta enlazada con la que va a establecer una conexión peering con la VPC debe estar habilitada para VRF. Para obtener más información sobre VRF, consulte [este documento](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud).

3. También debe añadir una ruta de vuelta a su VPC habilitada para clásico desde cada instancia que utilice en el lado de la infraestructura clásica.

## Restricciones
{: #classic-peering-restrictions}

Sólo se puede habilitar 1 VPC por región para el peering clásico.
